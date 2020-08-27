This is an 🌠experimental🌠 project. The shootings stars are for emphasis.

What if we used a rules engine to manage all the state of a frontend UI? Can a rules engine replace [reframe](https://github.com/Day8/re-frame)? Let's find out, children!

There is prior art here, including [precept](https://github.com/CoNarrative/precept), which tried to use [Clara Rules](https://github.com/cerner/clara-rules) for this purpose. But this time I'm using a rules engine built for fast updates of facts: [O'Doyle Rules](https://github.com/oakes/odoyle-rules).

For this experiment, I'm using [Rum](https://github.com/tonsky/rum) as my React wrapper, because it is flexible and is great at server-side rendering. The goal is to define Rum components as *rules* and make them automatically update when the data required by the rules is updated.

Here's a button that displays how many times it's been clicked:

```clojure
(require
  '[rum.core :as rum]
  '[odoyle.rules :as o]
  '[odoyle.rum :as orum])

(def components
  (orum/ruleset
    {click-counter
     [:what
      [::global ::clicks clicks]
      :then
      (let [*session (orum/prop)]
        [:button {:on-click (fn [_]
                              (swap! *session
                                (fn [session]
                                  (-> session
                                      (o/insert ::global ::clicks (inc clicks))
                                      o/fire-rules))))}
         (str "Clicked " clicks " " (if (= 1 clicks) "time" "times"))])]}))

(def *session
  (-> (reduce o/add-rule (o/->session) components)
      (o/insert ::global ::clicks 0)
      o/fire-rules
      atom))

(rum/mount (click-counter *session) (js/document.querySelector "#app"))
```

The `click-counter` rule generates a var holding a valid Rum component. When the values in the `:what` block are updated, the rule re-fires, which causes the component to update.

I'm converting [Dynadoc's UI](https://github.com/oakes/Dynadoc/blob/master/src/dynadoc/common.cljc) to use this system to see how it works in a larger project.

## Development

* Install [the Clojure CLI tool](https://clojure.org/guides/getting_started#_clojure_installer_and_cli_tools)
* To develop with figwheel: `clj -A:dev`
* To install the release version: `clj -A:prod install`
