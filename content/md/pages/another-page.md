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
  [:div {:style {:height "200px" :max-height "200px" :overflow "auto"}
         :id "fib-results"}
    [:p
      (clojure.string/join "\n" (map str @fib-results))]])
```

```klipse-cljs
(require-macros '[cljs.core.async.macros :refer [go go-loop]])
(require '[cljs.core.async :as async])

(defn controllable-channel [control-channel input-channel]
  "Creates a new channel that copies elements from input-channel
   as long as there are items in control-channel. One can control
   the timing and number of items moving from input-channel to
   ouput-channel by controlling the presence of items on the
   control-channel. The particular values on the control-channel
   are ignored. Once either control-channel or input-channel closes
   then the resulting channel will also close."
  (async/map (fn [x y] y) [control-channel input-channel]))

(def system-state (r/atom {}))

(defn initialize-state [num-items]
  (let [fib-input (async/chan)
        fib-control (async/chan)
        fib-output (async/map
                      (fn [n] {:input n, :result (fib n)})
                      [(controllable-channel fib-control fib-input)])]
    (reset! system-state {
      :fib-input fib-input
      :fib-control fib-control
      :fib-output fib-output
      :events (async/chan)
      ; Either :idle or :busy
      :computation-state :idle
      ; Either :stopped or :running or :shutdown
      :app-status :stopped})
    (reset! fib-results [])
    (async/onto-chan fib-input (range num-items))))

(defn stopped->running []
  (swap! system-state assoc :app-status :running)
  (when (= :idle (:computation-state @system-state))
    (swap! system-state assoc :computation-state :busy)
    (go
      (async/>! (:fib-control @system-state) :run-next))))

(defn running->stopped []
  (swap! system-state assoc :app-status :stopped))

(defn handle-play-pause [event]
  (case (:app-status @system-state)
    :stopped (stopped->running)
    :running (running->stopped)))

(defn process-result [{n :input, fib-n :result}]
  (swap! fib-results conj fib-n)
  (if (= :running (:app-status @system-state))
    (go
      ; We need this timeout so the computational side of the
      ; system will sleep for a little, giving the UI side a
      ; little access to the CPU to process user input.
      (async/<! (async/timeout 100))
      (async/>! (:fib-control @system-state) :run-next))
    (swap! system-state assoc :computation-state :idle)))

(defn run-loop [num-items]
  (initialize-state num-items)
  (go-loop []
    (if-let [result (async/<! (:fib-output @system-state))]
      (do
        (process-result result)
        (recur))
      ; If the result is nil that means we've run out of inputs.
      (swap! system-state assoc :app-status :shutdown))))

(def num-iterations 40)

(defn reset-system []
  (run-loop num-iterations))

(run-loop num-iterations)
```

```klipse-reagent
(defn play-pause-button [app-status]
  [:button {:on-click handle-play-pause}
    (if (= app-status :running)
      [:i {:class "fa fa-pause-circle" :style {:color "red" :font-size "200%"}}]
      [:i {:class "fa fa-play-circle" :style {:color "green" :font-size "200%"}}])])

; It looks like you need to have component definitions for the updating
; to work correctly, but I don't honestly know why.
(defn fib-output-component []
  [:div
    ; It's useful to put the button _above_ the output, otherwise it keeps
    ; getting pushed down the page and you end up having to chase it it you
    ; want to stop the thing.
    (when-not (= :shutdown (:app-status @system-state))
      [play-pause-button (:app-status @system-state)])
    (when (not-empty @system-state)
      [:button
        {:on-click reset-system :style {:color "red" :font-size "150%"}}
        "Reset"])
    [fib-results-component]])

[fib-output-component]
```
