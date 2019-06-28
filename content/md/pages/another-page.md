{:title "Another Page"
 :layout :page
 :page-index 1
 :klipse true}

## Look at this sweet, sweet page

this is another custom page
totally not a post

```klipse-cljs
(defn fib
  [n]
  (if (< n 2)
    n
    (+ (fib (- n 1)) (fib (- n 2)))))
```

```klipse-cljs
(require '[reagent.core :as r])
```

```klipse-cljs
(def fib-results (r/atom []))

(defn fib-results-component []
  [:div [:p (str @fib-results)]])

(def running (r/atom false))

(defn flip-status []
  (swap! running not))
```

```klipse-cljs
(require-macros '[cljs.core.async.macros :refer [go go-loop]])
(require '[cljs.core.async :as async])

(def c (async/chan))

(go-loop [n (async/<! c)]
  (when n
    (swap! fib-results conj (fib n))
    (recur (async/<! c))))

; It appears that you need at least *some* timeout/sleep/pause here
; for the UI to have the opportunity to update.
(go
  (loop [n 0]
    (if (and @running (< n 35))
      (do
        (async/>! c n)
        (async/<! (async/timeout 100))
        (recur (inc n)))
      (do
        (async/<! (async/timeout 100))
        (recur n)))))
```

```klipse-reagent
(defn play-pause-button [is-running]
  [:button {:on-click flip-status}
    (if is-running "Stop" "Start")])

; It looks like you need to have component definitions in a function in
; order to use the key metadata thing, and that seems critical for
; components updating properly.
(defn fib-output-component []
  [:div
    ; It's useful to put the button _above_ the output, otherwise it keeps
    ; getting pushed down the page and you end up having to chase it it you
    ; want to stop the thing.
    [play-pause-button @running]
    [fib-results-component]])

[fib-output-component]
```

<div>
  <pre id="run-output">Output will go here eventually…</pre>
</div>
