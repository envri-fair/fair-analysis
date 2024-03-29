http://130.60.24.146:7883/sparql # https://virtuoso.nps.petapico.org/sparql

https://github.com/peta-pico/dsw-nanopub-api/blob/main/make_matrix.rq

prefix fip: <https://w3id.org/fair/fip/terms/>
prefix dct: <http://purl.org/dc/terms/>
prefix dce: <http://purl.org/dc/elements/1.1/>
prefix npa: <http://purl.org/nanopub/admin/>
prefix npx: <http://purl.org/nanopub/x/>
prefix np: <http://www.nanopub.org/nschema#>
prefix dcat: <https://www.w3.org/ns/dcat#>

select distinct ?fip_index ?fip_title ?community ?c ?question ?q ?sort ?nochoice ?resource ?res ?reslabel ?rel ?resourcetype ?startdate ?enddate ?date where {
  ?np np:hasAssertion ?assertion .
  graph npa:graph { ?np dct:created ?date . }
  graph ?assertion {
    ?decl a fip:FIP-Declaration ;
      fip:refers-to-question ?question ;
      fip:declared-by ?community ;
      ?rel ?resource .
    optional { ?decl dcat:startDate ?startdate . }
    optional { ?decl dcat:endDate ?enddate . }
  }
  graph npa:graph { ?index_np npa:hasValidSignatureForPublicKey ?pubkey . }
  ?index_np np:hasAssertion ?index_a .
  ?index_np np:hasPublicationInfo ?index_i .
  graph ?index_a {
    ?fip_index npx:includesElement ?np .
  }
  graph ?index_i {
    ?fip_index dce:title ?fip_title_l .
  }
  filter not exists {
    ?fip_newer_index npx:includesElement ?newer_np ;
      dce:title ?fip_title_l .
    graph npa:graph { ?newer_np dct:created ?newer_date . }
    filter (?newer_date > ?date)
  }
  filter not exists {
    graph npa:graph {
      ?index_np_new npa:hasHeadGraph ?nh .
      ?index_np_new npa:hasValidSignatureForPublicKey ?pubkey .
    }
    graph ?nh {
      ?index_np_new np:hasPublicationInfo ?ni .
    }
    graph ?ni {
      ?index_np_new npx:supersedes ?index_np .
    }
  }
  filter not exists {
    graph npa:graph {
      ?index_np_r npa:hasHeadGraph ?rh .
      ?index_np_r npa:hasValidSignatureForPublicKey ?pubkey .
    }
    graph ?rh {
      ?index_np_r np:hasAssertion ?ra .
    }
    graph ?ra {
      ?somebody npx:retracts ?index_np .
    }
  }
  values ?rel {
    fip:declares-current-use-of fip:declares-planned-use-of fip:declares-planned-replacement-of
    # unofficial:
    fip:declares-replacement-from fip:declares-replacement-to
  }
  
optional {
  graph npa:graph { ?res_np npa:hasValidSignatureForPublicKey ?res_pubkey . }
  ?res_np np:hasAssertion ?res_a .
  graph ?res_a {
    ?resource rdfs:label ?resourcelabel .
  }
  filter not exists {
    graph npa:graph {
      ?res_np_new npa:hasHeadGraph ?rnh .
      ?res_np_new npa:hasValidSignatureForPublicKey ?res_pubkey .
    }
    graph ?rnh {
      ?res_np_new np:hasPublicationInfo ?rni .
    }
    graph ?rni {
      ?res_np_new npx:supersedes ?res_np .
    }
  }
  filter not exists {
    graph npa:graph {
      ?res_np_r npa:hasHeadGraph ?rrh .
      ?res_np_r npa:hasValidSignatureForPublicKey ?res_pubkey .
    }
    graph ?rrh {
      ?res_np_r np:hasAssertion ?rra .
    }
    graph ?rra {
      ?rsomebody npx:retracts ?res_np .
    }
  }
  optional {
    values ?resourcetype { fip:Available-FAIR-Enabling-Resource fip:FAIR-Enabling-Resource-to-be-Developed }
    ?resource a ?resourcetype
  }
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
