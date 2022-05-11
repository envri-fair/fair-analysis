https://virtuoso.nps.petapico.org/sparql

https://github.com/peta-pico/dsw-nanopub-api/blob/main/make_matrix.rq

prefix fip: <https://w3id.org/fair/fip/terms/>
prefix dct: <http://purl.org/dc/terms/>
prefix dce: <http://purl.org/dc/elements/1.1/>
prefix npa: <http://purl.org/nanopub/admin/>
prefix npx: <http://purl.org/nanopub/x/>
prefix np: <http://www.nanopub.org/nschema#>

select distinct ?p ?y ?t ?fip_title ?c ?q ?res 
where {
  ?np np:hasAssertion ?assertion .
  graph npa:graph { ?np dct:created ?date . }
  graph ?assertion {
    ?decl a fip:FIP-Declaration ;
      fip:refers-to-question ?question ;
      fip:declared-by ?community ;
      ?rel ?resource .
  }
  ?fip_index npx:includesElement ?np ;
    dce:title ?fip_title_l .
  filter not exists {
    ?fip_newer_index npx:includesElement ?newer_np ;
      dce:title ?fip_title_l .
    graph npa:graph { ?newer_np dct:created ?newer_date . }
    filter (?newer_date > ?date)
  }
  values ?rel {
    fip:declares-current-use-of fip:declares-planned-use-of fip:declares-planned-replacement-of
    # unofficial:
    fip:declares-replacement-from fip:declares-replacement-to
  }
  optional {?resource rdfs:label ?resourcelabel}
  optional {
    values ?resourcetype { fip:Available-FAIR-Enabling-Resource fip:FAIR-Enabling-Resource-to-be-Developed }
    ?resource a ?resourcetype
  }
  bind (replace(str(?community), ".*#", "") as ?c)
  bind (replace(str(?question), "^.*-([^-MD]+(-[MD]+)?)$", "$1") as ?q)
  bind (concat(replace(?q, "F|M", "0"), "x") as ?sort)
  bind (replace(str(?resource), "^.*?(#|/)([^/#]*/?[^/#]*)/?$", "$2") as ?res)
  bind (str(?resourcelabel) as ?reslabel)
  bind (str(?fip_title_l) as ?fip_title)
  bind (replace(str(?q), "^(.*)-(.*)$", "$1") as ?p)
  bind (replace(str(?q), "^(.*)-(.*)$", "$2") as ?t)
  bind (replace(str(?fip_title_l), "^(.*FIP[_|\\s])(.*)$", "$2") as ?y)
  bind ("" as ?nochoice)
  filter (?c != "ENVRI")
  filter (!strends(?fip_title, "2022$"))
} order by ?sort ?c
