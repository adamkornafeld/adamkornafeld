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
  background_color: '#0d164a'
  background_image:
    src: https://www.dalipaintings.com/assets/img/paintings/the-disintegration-of-the-persistence-of-memory.jpg
---

A while back I wrote about [how it all started](https://kornafeld.com/2021/10/01/how-it-all-started.html) - the tale of a university freshman armed with Pascal and an idea inspired by Dalí's melting clocks. Back then, I promised to recreate that dynamic clock and write about it. Well, here we are. The original Pascal implementation is lost to time - perhaps fitting for a project about time itself - but the concept lives on, now reborn in JavaScript and React.

Quick recap for those who haven't read the origin story: mechanical clocks have a fundamental limitation. The shape of the clock face is dictated by the length of the longest hand. But what if the hands could _breathe_, extending and contracting as they sweep around the face? Computers free us from the shackles of physical matter, allowing clock hands that dynamically adjust their length to fit any shape.

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

For each segment of our boundary polygon, we check if the ray from the center intersects it. We keep track of the closest intersection - that's how long our hand should be. The math here is good old linear algebra: solving for the intersection of a ray and a line segment.


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


🔘 Melting Camembert
--------------------

Dalí saw melting clocks in a piece of Camembert cheese warming in the sun. I saw them in the logical flexibility of computer graphics. Different inspirations, same liberation from the rigid march of mechanical time. And that wraps our coding session for today. Happy coding!
