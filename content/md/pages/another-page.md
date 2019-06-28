{:title "Another Page"
 :layout :page
 :page-index 1
 :klipse true}

## Look at this sweet, sweet page

this is another custom page
totally not a post

```klipse-cljs
(map #(* % %) (range 10))
```

Let's try `do-seq` with a `div`:

```klipse-cljs
(defn report
  [value]
  (let [previous-output (.-innerHTML (js/document.getElementById "run-output"))]
    (set!
      (.-innerHTML (js/document.getElementById "run-output"))
      (str previous-output "\n" value))))

(defn fib
  [n]
  (if (< n 2)
    n
    (+ (fib (- n 1)) (fib (- n 2)))))
```

```klipse-cljs
(loop [x 0]
  (if (<= x 10)
    (do
      (report (fib x))
      (recur (inc x)))))
```

```klipse-cljs
(require '[reagent.core :as r])
```

```klipse-cljs
; (require '[promesa.cljs.core :as p])
```

```klipse-reagent
(defn hello [name]
  [:p (str "Hello " name "!")])

[hello "Klipse"]
```

```klipse-reagent
;(def fib-results (r/atom []))

;(doseq [x (range 10)]
;  (swap! fib-results conj (fib x)))

;(loop [x 0]
;  (if (<= x 10)
;    (do
;      (js/setTimeout (swap! fib-results conj (fib x)) 1000)
;      (recur (inc x)))))

;[:p (str @fib-results)]

[:p "Silly nonsense"]
```

```klipse-cljs
(def fib-results (r/atom []))

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
