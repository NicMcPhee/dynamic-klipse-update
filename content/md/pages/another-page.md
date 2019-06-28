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
(def running (r/atom false))

(defn fib-results-component []
  ^{:key (count @fib-results)}
  [:div [:p (str @fib-results)]])
```

```klipse-cljs
(require-macros '[cljs.core.async.macros :refer [go go-loop]])
(require '[cljs.core.async :as async])

(def c (async/chan))

(go-loop [n (async/<! c)]
  (when n
    (swap! fib-results conj (fib n))
    (recur (async/<! c))))

(go
  (doseq [n (range 10)]
    (async/>! c n)
    ; It appears that you need at least *some* timeout/sleep/pause here
    ; for the UI to have the opportunity to update. Weirdly, though, 0
    ; is enough -- it's obvious the call to `async/timeout` that matters,
    ; not it's argument.
    (async/<! (async/timeout 0))))
```

```klipse-reagent
[fib-results-component]
```

<div>
  <pre id="run-output">Output will go here eventually…</pre>
</div>
