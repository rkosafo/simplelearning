<script>
  import {
    Grid, Row, Column,
    Tile
  } from 'carbon-components-svelte'
  import { createEventDispatcher } from 'svelte'
  export let videos = []

  let dispatch = createEventDispatcher()
  function onVideoClick(video){
    dispatch('play', video)
  }
</script>

<div>
  <Grid>
    <Row>
      {#each videos as video}
        <Column style="cursor: pointer" padding>
          <Tile on:click={() => onVideoClick(video)} style="width:100%; height:100%; text-align:center">{video.title}</Tile>
        </Column>
      {/each}
    </Row>
  </Grid>
</div>