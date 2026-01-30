---
layout: article
title: The one that brings the Dalí Clock to life
tags: javascript react clock UI Dalí Salvador pascal
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    src: https://media.npr.org/assets/img/2015/02/04/spiral-bbf98c485003ba03cc036a52dd4a4b068e96e965-s1600-c85.webp
---

When I was looking for a cover image for the [very first post](https://kornafeld.com/2021/04/10/on-naming.html) I almost instinctively reached out to one of my favorite paintings: [The Persistence of Memory](https://en.wikipedia.org/wiki/The_Persistence_of_Memory) by Dalí. You might know it by the name: The Melting Clocks. It's only now, years later, that it dawned on me why.

The story says Dalí got his inspiration for the painting from the surreal way he once saw a piece of runny Camembert cheese melting in the Sun. The melting clocks represent the _omnipresence_ of time, and identify its _mastery_ over human beings. I feel lucky to have been able to witness this painting in real life at the [MoMA](https://www.moma.org/) in New York City.


🎥 How it all started
---------------------

I am fascinated by the concept of time. Don't really know (yet) why, but I always have been. Back when I was a freshman at the university - Budapest University of Technology and Economics or [BME](https://www.bme.hu/?language=en) that is - the gateway drug language into software engineering they taught us was _Pascal_. I know, right?! I feel old. During the first semester everyone had to pick an idea and implement it as their take home assignment over the course of those few months. When people think about Pascal, command line applications usually first pop in mind. Not for me though. Excited to get my hands dirty with _real_ software engineering - after having been hacking at it on my own well before university - I wanted to jump in the deep.

So I picked a _graphics_ problem. An idea that I had for some time that found its roots in Dalí's painting and that I lacked the know-how on how to execute it up until that time. See, mechanical clocks, watches and timepieces have a major limitation in my eyes. The shape of them are mainly influenced by the length of their hour, minute and second hands. The longest hand sets the minimum of the radius the clock face can have. Anything smaller and the face would not be able to complete a full circle without bumping into the wall of the clock face. Or worse run off the clock face. Sure, you see rectangular clock faces every now and then. But that's usually the farthest the imagination of the creator goes. Enter Dalí. He was first to break down this barrier raised by the physical limitations presented by watch hands in his painting. However, he was cheating or more like lucky in a sense that through his painting he was able to freeze time so he didn't have to worry about the length of the clock hands.


⏰ Free-style clock face
------------------------

For the take home assignment my idea was to take Dalí's concept one notch further leveraging the creative freedom provided to us by computers. What if the hands could _breathe_, extending and contracting as they sweep around the face? The original Pascal implementation is lost to time - perhaps fitting for a project about time itself - but the concept lives on, now reborn in JavaScript and React.

The algorithm is deceptively simple:
- Draw an arbitrary closed shape
- Place the center point of the clock somewhere inside that shape  
- For each hand, calculate the distance from the center to the boundary in the direction the hand is pointing
- Draw the hand with that calculated length
- Repeat sixty times per second

Let's dig into the implementation, shall we?


📐 Ray Casting
--------------

The heart of the algorithm is ray casting. Given a point (the center) and an angle (where the hand is pointing), we need to find where a ray in that direction intersects our custom boundary. Here's the core function:

```javascript
function getIntersectionDistance(center, angle, boundaryPoints) {
  const radians = (angle - 90) * Math.PI / 180;
  const direction = { x: Math.cos(radians), y: Math.sin(radians) };
  
  let minDistance = Infinity;
  
  for (let i = 0; i < boundaryPoints.length - 1; i++) {
    const p1 = boundaryPoints[i];
    const p2 = boundaryPoints[i + 1];
    
    const distance = raySegmentIntersection(center, direction, p1, p2);
    if (distance !== null && distance < minDistance) {
      minDistance = distance;
    }
  }
  
  return minDistance;
}
```

For each segment of our boundary polygon, we check if the ray from the center intersects it. We keep track of the closest intersection - that's how long our hand should be. The math here is good old linear algebra: solving for the intersection of a ray and a line segment. You might be more familiar with ray casting's flashier cousin: [ray tracing](https://en.wikipedia.org/wiki/Ray_tracing_(graphics)). While ray tracing follows rays as they bounce around a scene to simulate realistic lighting and reflections, ray casting is simpler - we just need to find the first thing a ray hits and stop there.


🎯 Drawing the Boundary
-----------------------

What makes this implementation interactive is the ability to draw your own clock face. Users can sketch any closed shape they like, and the clock adapts. The drawing logic tracks mouse or touch events to collect points as the user draws, then validates that the shape forms a proper closed loop around the clock center:

```javascript
function useBoundaryDrawing() {
  const [drawingPoints, setDrawingPoints] = useState([]);
  const [isDrawing, setIsDrawing] = useState(false);
  const startPoint = useRef(null);

  const startDrawing = (point, center) => {
    if (distance(point, center) < MIN_DISTANCE) return;
    startPoint.current = point;
    setDrawingPoints([point]);
    setIsDrawing(true);
  };

  const continueDrawing = (point, center) => {
    if (!isDrawing) return;
    // Keep points outside the minimum radius from center
    if (distance(point, center) < MIN_DISTANCE) {
      const angle = Math.atan2(point.y - center.y, point.x - center.x);
      point = {
        x: center.x + Math.cos(angle) * MIN_DISTANCE,
        y: center.y + Math.sin(angle) * MIN_DISTANCE
      };
    }
    setDrawingPoints(prev => [...prev, point]);
  };

  const endDrawing = (center) => {
    if (!isDrawing || !startPoint.current) return;
    const lastPoint = drawingPoints[drawingPoints.length - 1];
    // Close the loop if end point is near start point
    if (drawingPoints.length >= MIN_POINTS && 
        distance(lastPoint, startPoint.current) <= CLOSE_THRESHOLD) {
      const closedShape = [...drawingPoints, startPoint.current];
      if (isPointInsidePolygon(center, closedShape)) {
        createBoundary(closedShape);
      }
    }
    setIsDrawing(false);
    setDrawingPoints([]);
  };

  return { startDrawing, continueDrawing, endDrawing, drawingPoints, isDrawing };
}
```

The key validation here is `isPointInsidePolygon` - we only accept shapes that actually contain the clock center. Drawing a shape that excludes the center would result in hands pointing into the void, which wouldn't make much sense for a clock.


🔄 Caching for Performance
-------------------------

One optimization worth mentioning: recalculating ray intersections for every angle, every frame would be wasteful. Instead, when a boundary is drawn, we pre-calculate the maximum hand length for all 360 integer degrees and store them in a Map:

```javascript
function calculateAngleToRadius(points, center) {
  const angleToRadius = new Map();
  
  for (let angle = 0; angle < 360; angle++) {
    const radius = getIntersectionDistance(center, angle, points);
    angleToRadius.set(angle, radius);
  }
  
  return angleToRadius;
}
```

At runtime, we interpolate between adjacent cached values for smooth animation. This turns an O(n) operation per hand per frame into a simple Map lookup with some basic math.


🖱️ Try It Yourself
------------------

The best way to understand this clock is to play with it. Head over to [kornafeld.com/clock](https://kornafeld.com/clock/) and draw your own clock face. Sketch a star, a heart, or Dalí's melting blob itself. Drag the center point around and watch the hands stretch and shrink to fill the space.

What fascinates me about this project is how an idea conceived in a Pascal assignment decades ago finds new life in modern web technologies. The core algorithm hasn't changed - it's still ray casting and intersection math. But the delivery mechanism has transformed from a university assignment running on a local machine to an interactive web experience accessible to anyone with a browser.


🔘 Connecting the dots
----------------------

It is indeed funny how life gives you dots and it's up to you to connect them. _Or not_. This is me having a _great time_ realizing how I subconsciously used Dalí's painting as the cover of the first post of my blog in which I focus _mainly_ on software engineering. Only to realize later that how that very painting ties back into the early days of my software endeavors at BME.

Also fascinating that it seems to me that this idea has not been leveraged much yet with the dawn of all the smart watches out there that have the necessary screens built in to support watch faces of dynamic shape. A missed opportunity?!

Dalí saw melting clocks in a piece of Camembert cheese warming in the sun. I saw them in the logical flexibility of computer graphics. Different inspirations, same liberation from the rigid march of mechanical time. And that wraps our coding session for today. Happy coding!
