(ns client-information.governor
  "ClientInformationGovernor — the independent safety/traceability
  layer for the ISCO-08 4229 independent client-information-services
  actor. Wired as its own `:govern` node in
  `client-information.actor`'s StateGraph, downstream of `:advise` —
  the Advisor has no notion of client provenance or disclosure/
  emergency-routing risk, so this MUST be a separate system able to
  reject a proposal (itonami actor pattern, per ADR-2607011000 /
  CLAUDE.md Actors section).

  `check` is a pure function of (request, context, proposal, store) ->
  verdict; it never mutates the store. The StateGraph's `:decide` node
  routes on the verdict:
    :hard? true                → :hold  (irreversible, no write)
    :escalate? true            → :request-approval (interrupt-before)
    otherwise                  → :commit

  HARD invariants (:hard? true, ALWAYS :hold, never overridable):
    1. client provenance  — the request's client must be registered.
    2. no-actuation         — proposal :effect must be :propose.
  ESCALATION invariants (:escalate? true, ALWAYS human sign-off, per the
  README robotics-premise: disclosing sensitive client information
  and emergency-request routing always require human sign-off):
    3. :op :disclose-sensitive-information.
    4. :op :route-emergency-request.
    5. low confidence (< `confidence-floor`)."
  (:require [client-information.store :as store]))

(def confidence-floor 0.6)
(def ^:private escalating-ops #{:disclose-sensitive-information :route-emergency-request})

(defn- hard-violations [{:keys [proposal]} client-record]
  (cond-> []
    (nil? client-record)
    (conj {:rule :no-client :detail "未登録 client"})

    (not= :propose (:effect proposal))
    (conj {:rule :no-actuation :detail "effect は :propose のみ許可（直接書込禁止）"})))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `client-information.store/Store`. Returns
  `{:ok? bool :violations [...] :confidence n :hard? bool :escalate? bool}`."
  [request context proposal store]
  (let [client-record (store/client store (:client-id request))
        hard (hard-violations {:proposal proposal} client-record)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        risky-op? (contains? escalating-ops (:op proposal))]
    {:ok? (and (not hard?) (not low?) (not risky-op?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? risky-op?))}))
