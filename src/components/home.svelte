<script>
  import {
    Header, HeaderUtilities, HeaderAction, HeaderSearch,
    HeaderPanelLink, HeaderPanelLinks, HeaderPanelDivider,
    SkipToContent, Content,
    Grid, Row, Column,
    SideNav, SideNavItems, SideNavLink,
    Search,
    Breadcrumb, BreadcrumbItem,
    InlineNotification
  } from 'carbon-components-svelte'
  import {
    flow, map, uniq, sortBy, identity, filter, find
  } from 'lodash/fp'
  import Theme from './Theme.svelte'
  import PlayList from './playlist.svelte'
  import { onMount } from 'svelte'

  let config =
    {
      "platform": "GRA",
      "gettingStartVideoUrl": "",
      "defaultRole": "",
      "allowGlobalVideoSearch": true,
      "allowRoleVideoSearch": true
    }
  let theme = 'white'
  let isSideNavOpen = false
  let switcherOpen = false
  let allVideos = []
  let roles = []
  let activeRole = ""
  let roleVideos = []
  let filteredRoleVideos = []
  let roleSearchValue = ""
  let headerSearchValue = ""

  let searchData = []
  let filteredSearchData = []
  let searchActive = false

  let activeVideo =
    { url: "",
      title: "",
      description: "" }

  onMount(async () => {
    //get the configuration
    const configResponse = await fetch("/config.json")
    const configData = await configResponse.json()
    config = configData
    activeVideo =
      { url: config.gettingStartVideoUrl,
        title: "Getting Started",
        description: "" }
    //get the videos
    const response = await fetch("/videos.json")
    allVideos = await response.json()
    roles = flow(
      map(x => x.role),
      uniq,
      sortBy(identity)
    ) (allVideos)

    searchData = flow(
      map(x => {
        return { text: x.title, description: x.role, ref: x }
      }),
      sortBy(x => x.text)
    ) (allVideos)
  })

  function activateRole(role) {
    isSideNavOpen = false
    activeRole = role;
    roleVideos = flow (
      filter (x => x.role === role)
    ) (allVideos)
    filteredRoleVideos = roleVideos
    roleSearchValue = ""
  }

  function filterRoleVideos(roleSearchValue) {
    if (!roleSearchValue){
      filteredRoleVideos = roleVideos;
      return;
    }
    filteredRoleVideos = filter (x => x.title.toLowerCase().indexOf(roleSearchValue.toLowerCase()) > -1) (roleVideos)
  }

  function playVideo(video){
    activeVideo =
      { url: video.url,
        title: video.title,
        description: video.role || activeRole }
  }

  function playVideoFromSearch(e){
    if (!e || !e.detail) return
    var video = e.detail.selectedResult.ref
    if (!video){
      return
    }
    playVideo(video)
  }

  $: filterRoleVideos (roleSearchValue)
  $: filteredSearchData =
    !!headerSearchValue
      ? searchData.filter(x => {
        return x.text.toLowerCase().includes(headerSearchValue.toLowerCase())
            || x.description.toLowerCase().includes(headerSearchValue.toLowerCase())
      })
      : searchData

</script>

<svelte:head>
  <title>{config.title}</title>
</svelte:head>


<Theme {theme}>
<Header company={config.company} platformName={config.platform} bind:isSideNavOpen persistentHamburgerMenu={true}>
   <div slot="skip-to-content">
     <SkipToContent />
   </div>
   <HeaderUtilities>
     {#if config.allowGlobalVideoSearch}
      <HeaderSearch placeholder="Find Videos" bind:value={headerSearchValue} results={filteredSearchData} bind:active={searchActive} on:select={playVideoFromSearch}/>
     {/if}
     <HeaderAction bind:isOpen={switcherOpen}>
       <HeaderPanelLinks>
         <HeaderPanelDivider>Themes</HeaderPanelDivider>
         <HeaderPanelLink on:click={() => theme = "white"}>White</HeaderPanelLink>
         <HeaderPanelLink on:click={() => theme = "g10"}>Light Gray</HeaderPanelLink>
         <HeaderPanelLink on:click={() => theme = "g90"}>Gray</HeaderPanelLink>
         <HeaderPanelLink on:click={() => theme = "g100"}>Black</HeaderPanelLink>
       </HeaderPanelLinks>
     </HeaderAction>
   </HeaderUtilities> 

   <SideNav bind:isOpen={isSideNavOpen}>
     <SideNavItems>
       {#each roles as role}
        <SideNavLink text={role} on:click={_ => activateRole(role)}/>
       {/each}
     </SideNavItems>
   </SideNav>
 </Header>

 <Content>
   <Grid>
     <Row>
       <Column style="text-align: center">
         {#if activeVideo && activeVideo.url}
          <h3>{activeVideo.title}</h3>
          <h5>{activeVideo.description}</h5>
          {#key activeVideo.url}
            <video controls autoplay>
              <source src={activeVideo.url} type="video/mp4" />
              <track kind="captions"/>
                Sorry, your browser does not support embeded videos
            </video>
          {/key}
         {:else}
           <p>No video selected</p>
         {/if}
       </Column>
     </Row>
     <Row class="videos">
       <Column>
       {#if activeRole}
         <Grid narrow>
           <Row>
             <Column sm={4} md={8} lg={10} style="margin-top: 16px">
                <Breadcrumb noTrailingSlash>
                  <BreadcrumbItem>{activeRole}</BreadcrumbItem>
                  <BreadcrumbItem isCurrentPage>videos</BreadcrumbItem>
                </Breadcrumb>
              </Column>
             <Column>
              {#if config.allowRoleVideoSearch}
                <Search placeholder={`Find ${activeRole} Videos`} bind:value={roleSearchValue} />
              {/if}
             </Column>
           </Row>
         {#if filteredRoleVideos && filteredRoleVideos.length}
          <div class="video-list">
            <PlayList videos={filteredRoleVideos} on:play={e => playVideo(e.detail)}/>
          </div>
        {:else}
        <div>
          <!-- report that component fails to add all classes when additional classes are specified -->
          <InlineNotification hideCloseButton class="center bx--inline-notification--warning bx--inline-notification bx--inline-notification--hide-close-button"
            kind = "warning"
            title = "There are no videos to show. Refine your search" />
        </div>
        {/if}
         </Grid>
       {:else}
          <div>
            <InlineNotification hideCloseButton class="center bx--inline-notification--warning bx--inline-notification bx--inline-notification--hide-close-button"
              kind = "warning"
              title = "No role selected. Select a role first" />
          </div>
       {/if}
      </Column>
     </Row>
   </Grid>
 </Content>
</Theme>


<style>
  video {
    width: 75%;
    max-width: 800px;
  }
  :global(.center) {
    margin-left: auto;
    margin-right: auto;
  }
</style>