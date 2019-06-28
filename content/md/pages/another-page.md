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

(def fib-input (async/chan))
(def fib-output
  (async/map fib [fib-input]))

; By having this wait until it gets a response to the previous
; "request" before adding the next value of `n` to the input
; channel we don't need to timeouts anymore.
(go
  (loop [n 0]
    (if (>= n 35)
      (async/close! fib-input)
      (if @running
        (do
          (async/>! fib-input n)
          (let [result (async/<! fib-output)]
            (swap! fib-results conj result))
          (recur (inc n)))
        (do
          (recur n))))))
```

```klipse-reagent
(defn play-pause-button [is-running]
  [:button {:on-click flip-status}
    (if is-running
      [:i {:class "fa fa-pause-circle" :style {:color "red" :font-size "200%"}}]
      [:i {:class "fa fa-play-circle" :style {:color "green" :font-size "200%"}}])])

; It looks like you need to have component definitions for the updating
; to work correctly, but I don't honestly know why.
(defn fib-output-component []
  [:div
    ; It's useful to put the button _above_ the output, otherwise it keeps
    ; getting pushed down the page and you end up having to chase it it you
    ; want to stop the thing.
    [play-pause-button @running]
    (if @running [:button {:on-click flip-status} [:i {:class "fa fa-stop-circle" :style {:color "red" :font-size "200%"}}]])
    [fib-results-component]])

[fib-output-component]
```

<div>
  <pre id="run-output">Output will go here eventually…</pre>
</div>
