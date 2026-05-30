<script>
  import P5 from 'p5-svelte';

  const sketch = (p) => {
    p.setup = () => {
      p.createCanvas(600, 400);
      p.background(20);
    };
    p.draw = () => {
      p.background(20, 20, 40);
      p.fill(100, 200, 255);
      p.noStroke();
      p.ellipse(p.mouseX, p.mouseY, 30, 30);
    };
  };
</script>

<h1 class="text-2xl font-bold p-4">TkNeuro Viz</h1>
<P5 {sketch} />