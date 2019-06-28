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

The point here is to try to print, _very slowly_, the fibonacci terms in the `div` below:

```klipse-cljs
(require '[reagent.core :as r])

(def report-string (atom "working"))

(defn report-now
  []
  (let [report-div (js/document.getElementById "run-output")]
    (set!
      (.-innerHTML report-div)
      @report-string)))

(defn fib
  [n]
  (if (< n 2)
    n
    (+ (fib (- n 1)) (fib (- n 2)))))
```

```klipse-cljs
(do
  (report-now)
  (loop [x 0]
    (if (<= x 30)
      (do
        (swap! report-string #(print-str % (fib x)))
        (report-now)
        (recur (inc x))
        ))))
```


<div>
  <pre id="run-output">Output will go here eventually…</pre>
</div>
