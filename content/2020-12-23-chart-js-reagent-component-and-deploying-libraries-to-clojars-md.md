---
title: chart.js reagent component and deploying libraries to clojars
date: 2020-12-23T22:10:33.168Z
cover: /assets/clojure.jpg
slug: reagent-component-clojars-deploy
category: dev
tags:
  - clojure
  - clojars
  - react
  - reagent
  - chart.js
  - lein
---
## Introduction

Recently I was integrating some charts with [chart.js](https://chartjs.org) in my [clojurescript](https://clojurescript.org)/[reagent](http://reagent-project.github.io) application and I came across this neat little snippet in stackoverflow which I thought was worth wrapping into a new library.

I did not dig much deeper in search of an existing library that does this as I have been looking for an opportunity to  implement and deploy a library to [Clojars](https://clojars.org) just to become familiar with the process.

So anyway, here is the fancy snippet:

```clojure
(ns chart-cljs.core
  (:require [reagent.core :as reagent]
            ["chart.js"]))

(defn- show-chart-fn [canvas-id chart-data]
  (fn []
    (let [ctx        (.. js/document
                         (getElementById canvas-id)
                         (getContext "2d"))]
          
      (js/Chart. ctx (clj->js chart-data)))))

(defn ^:export chart-component [chart-data]
  (let [canvas-id  (str (gensym))
        show-chart (show-chart-fn canvas-id chart-data)]
    (reagent/create-class
     {:component-did-mount #(show-chart)
      :display-name        (str "chart-cljs-component-" canvas-id)
      :reagent-render      (fn []
                             [:canvas {:id canvas-id}])})))
```

## Project Setup

I created the project using [shadow-cljs](https://shadow-cljs.org) running `shadow-cljs init` and added the most recent version of reagent as a dependency. Here is the `shadow-cljs.edn` file:

```clojure
;; shadow-cljs configuration
{:source-paths
 ["src/dev"
  "src/main"
  "src/test"]

 :dependencies
 [[reagent "1.0.0-rc1"]]

 :builds
 {:app {:target :browser
        :output-dir "public/js"
        :modules {:main {:entries [chart-cljs.core]}}}}}
```

## Generating JAR and POM

Reading through shadow-cljs docs, it is mentioned that the recommended/easiest way of deploying a cljs lib to clojars is with [Leiningen](https://www.leiningen.org). In order to use lein with our project we need to create a project.clj file in the root folder.

Here is mine:

```clojure
(defproject baskeboler/chart-cljs "1.0.2"
  :description "reagent component for chart.js"
  :url "https://github.com/baskeboler/chart.cljs"

  ;; this is optional, add what you want or remove it
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}

  :dependencies
  ;; always use "provided" for Clojure(Script)
  [[org.clojure/clojurescript "1.10.520" :scope "provided"]
   [reagent "1.0.0-rc1"]]

  :source-paths ["src/main"]
  :repositories
  {"clojars" {:url "https://clojars.org/repo"
              :sign-releases false
              :username "baskeboler"  ; <- this is being ignored
              :password :env/CLOJARS_TOKEN}} ; <- ignored also
  :deploy-repositories [["releases" :clojars] ["snapshots" :clojars]]          )
 
```

I signed up for a user in [clojars](https://clojars.org) After signing up you need to create a deploy token in the screen below. This token is what you need to use as password when running `lein deploy clojars` along with your clojars user name.

![image](/assets/deploytokens.png)

## References


* The code is hosted in [github](http://github.com/baskeboler/chart.cljs)
* My toy karaoke application using the new dependency is [here](https://github.com/baskeboler/cljs-karaoke-client)
* [Clojars](https://clojars.org)
