# Activity

- use Activity to hide part of your application

```tsx
<Activity mode={isShowingSidebar ? "visible" : "hidden"}>
  <Sidebar />
</Activity>
```

When an Activity boundary is **hidden** :

- React will visually hide its children using the display: "none" CSS property.
- It will also destroy their Effects
- cleaning up any active subscriptions.

 <!-- albeit : ان كان ذلك -->
 <!-- discarding : التخلص من -->
 <!-- discrete : منفصل -->

> While hidden, children still re-render in response to new props, albeit at a lower priority than the rest of the content.

When the boundary becomes visible again

 <!-- reveal : يكشف -->

- React will reveal the children with their previous state restored
- re-create their Effects.

Rather than completely **discarding content** that’s likely to become visible again, you can
use **Activity** to maintain and restore that content’s UI and internal state, while ensuring
that your hidden content has no unwanted side effects.

## Props

- children: The UI you intend to show and hide.
- mode: A string value of either 'visible' or 'hidden'. If omitted, defaults to 'visible'.

---

## Usage

- Restoring the state of hidden components
- Restoring the DOM of hidden components like [when write the form ,input]
- Pre-rendering content that’s likely to become visible
- Speeding up interactions during page load

(Activity does not detect data that is fetched inside an Effect.)

---

### Speeding up interactions during page load

- React includes an under-the-hood performance optimization called Selective Hydration.

#### Selective Hydration

- it works by hydrating your app’s initial HTML in chunks, enabling some components to become
  interactive even if other components on the page haven’t loaded their code or data yet.

#### Suspense boundaries

- it participate in Selective Hydration,because they naturally divide your component
  tree into units that are independent from one another
- it would also change the UI, since the Placeholder fallback would be displayed on
  the initial render

  Since Activity boundaries show and hide their children, they already naturally
  divide the component tree into independent units

## troubleshooting

- My hidden components have unwanted side effects
  - example video

The video and audio continue to play even after it’s been hidden,
because the tab’s <video> element is still in the DOM.
To fix this, we can add an Effect with a cleanup function that pauses the video:

```tsx
export default function VideoTab() {
  const ref = useRef();

  useLayoutEffect(() => {
    const videoRef = ref.current;

    return () => {
      videoRef.pause();
    };
  }, []);

  return <video ref={ref} controls playsInline src="..." />;
}
```

- When an <Activity> is “hidden”, all its children’s Effects are cleaned up. Conceptually,
  the children are unmounted, but React saves their state for later.

---
