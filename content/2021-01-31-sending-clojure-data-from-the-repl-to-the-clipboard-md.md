---
title: Sending clojure data from the REPL to the clipboard
date: 2021-01-31T20:50:37.435Z
cover: /assets/clojureavatar.png
slug: /pretty-print-clojure-data-to-clipboard
category: clojure
tags:
  - clojure
  - repl
  - clipboard
  - pprint
---
I wrote these utility functions so I could easily pprint clojure data structs into the clipboard.

```clojure
(ns clipboard.core
  (:require [clojure.pprint :refer [pprint]])
  (:import (java.awt.datatransfer DataFlavor Transferable StringSelection)
           (java.awt Toolkit)
           (java.io StringWriter))
           
(defn get-clipboard
  []
  (-> (Toolkit/getDefaultToolkit)
      (.getSystemClipboard)))

(defn slurp-clipboard
  []
  (when-let [^Transferable clip-text (some-> (get-clipboard)
                                             (.getContents nil))]
    (when (.isDataFlavorSupported clip-text DataFlavor/stringFlavor)
      (->> clip-text 
           (#(.getTransferData % DataFlavor/stringFlavor))
           (cast String)))))

(defn spit-clipboard
  [s]
  (let [sel (StringSelection. s)]
    (some-> (get-clipboard)
            (.setContents sel sel))))        

; an alias for spit
(def  str->clipboard spit-clipboard)

(defn  pprint-data-to-clipbaord 
  "pretty prints a data structure into the clipboard"
  [d]
  (let [wr (java.io.StringWriter.)]
    (pprint d wr)
    (str->clipboard (.toString wr))))

(comment 
  (def my-data {:a 1 :b 2}) ;; a clojure map
  (pprint-data-to-clipboard my-data) ;; pprint the map into the clipboard!
    

```
