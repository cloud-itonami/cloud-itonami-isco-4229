(ns client-information.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [client-information.store :as store]
            [client-information.governor :as governor]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-client! st {:client-id "client-1" :name "Acme Retail"})
    st))

(deftest ok-on-clean-respond
  (let [st (fresh-store)
        proposal {:op :respond :effect :propose :confidence 0.9 :stake :low}
        v (governor/check {:client-id "client-1"} {} proposal st)]
    (is (:ok? v))
    (is (not (:hard? v)))
    (is (not (:escalate? v)))))

(deftest hard-on-unregistered-client
  (let [st (fresh-store)
        proposal {:op :respond :effect :propose :confidence 0.9 :stake :low}
        v (governor/check {:client-id "no-such-client"} {} proposal st)]
    (is (:hard? v))
    (is (some #(= :no-client (:rule %)) (:violations v)))))

(deftest hard-on-no-actuation-violation
  (let [st (fresh-store)
        proposal {:op :respond :effect :direct-write :confidence 0.9 :stake :low}
        v (governor/check {:client-id "client-1"} {} proposal st)]
    (is (:hard? v))
    (is (some #(= :no-actuation (:rule %)) (:violations v)))))

(deftest escalates-on-sensitive-information-disclosure
  (let [st (fresh-store)
        proposal {:op :disclose-sensitive-information :effect :propose :confidence 0.9 :stake :high}
        v (governor/check {:client-id "client-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest escalates-on-emergency-request-routing
  (let [st (fresh-store)
        proposal {:op :route-emergency-request :effect :propose :confidence 0.9 :stake :high}
        v (governor/check {:client-id "client-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest escalates-on-low-confidence
  (let [st (fresh-store)
        proposal {:op :respond :effect :propose :confidence 0.2 :stake :low}
        v (governor/check {:client-id "client-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest store-records-and-ledger-append-only
  (let [st (fresh-store)]
    (store/commit-record! st {:client-id "client-1" :op :log})
    (store/append-ledger! st {:disposition :commit})
    (is (= 1 (count (store/records-of st "client-1"))))
    (is (= 1 (count (store/ledger st))))))
