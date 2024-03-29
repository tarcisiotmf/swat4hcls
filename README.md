# Data in use for Alzheimer disease study: combining gene expression, orthology, bioresource and disease datasets

## How to cite this work?

Mendes de Farias T, Kushida T, Sima AC,Dessimoz C,Chiba H, Bastian F, Masuya H (2023, February). Data in use for Alzheimer disease study: combining gene 
expression, orthology, bioresource and disease datasets. In Proceedings of the 14th International Semantic Web Applications and Tools for Healthcare and Life Sciences (SWAT4HCLS) Conference (Vol. 3415). CEUR-WS. 

## Introduction 

This repository contains the supplementary material to the [short paper](SWAT4HCLS_2023___Data_in_use_for_Alzheimer.pdf).

## Supplementary material description

The queries below can be executed at the SPARQL endpoint https://knowledge.brc.riken.jp/sparql.

**Table 1.** The query execution time of the federated vs centralised approaches. The queries were executed 10 times each at https://knowledge.brc.riken.jp/sparql.
| Query approach | Mean (seconds) | Standard deviation | Number of results |  
|---|---|---|---|
| [centralised](centralised_query.rq) | 25.93 | 0.91 | 8 |
| [federated](federated_query.rq) | 59.98 | 6.78 | 20 |

## Federated query
**Expected runtime: 60 seconds**

[SPARQL query file](federated_query.rq) 
```
PREFIX brso: <http://purl.jp/bio/10/brso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX lscr: <http://purl.org/lscr#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX genex: <http://purl.org/genex#>
PREFIX umls: <http://linkedlifedata.com/resource/umls/id/>

SELECT DISTINCT ?mouse ?homepage_mouse ?ensembl2
WHERE {
  SERVICE <https://bgee.org/sparql/> {
    ?oma_gene2 a orth:Gene .
    ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2 .
    ?oma_gene2 orth:organism/obo:RO_0002162 taxon:9606 . # human
    ?oma_gene2 genex:isExpressedIn ?cond .
    ?cond genex:hasAnatomicalEntity obo:UBERON_0000451 . # prefrontal cortex
    ?expr genex:hasExpressionCondition ?cond .
    ?expr genex:hasSequenceUnit ?oma_gene2 .
    ?expr a genex:Expression .
    ?expr genex:hasConfidenceLevel obo:CIO_0000029 . # high confidence level
    ?expr genex:hasExpressionLevel ?exprLevel .
    FILTER (?exprLevel > 99)
  }
  {
    SELECT DISTINCT ?mouse ?homepage_mouse ?ensembl2
    WHERE {
      GRAPH <http://metadb.riken.jp/db/xsearch_animal_brso> {
        ?mouse brso:genomic_feature/brso:has_genomic_segment/rdfs:seeAlso ?mgi1 .
        ?mouse foaf:homepage ?homepage_mouse .
      }
      GRAPH <http://metadb.riken.jp/db/mgi_ncbi_ensembl> {
        ?mgi1 rdfs:seeAlso ?ensembl1 .
      }
      GRAPH <http://metadb.riken.jp/db/omaRDF> {
        ?cluster a orth:OrthologsCluster .
        ?cluster orth:hasHomologousMember ?node1 .
        ?cluster orth:hasHomologousMember ?node2 .
        FILTER (?node1 != ?node2)
        ?node1 orth:hasHomologousMember* ?protein1 .
        ?node2 orth:hasHomologousMember* ?protein2 .
        ?protein1 sio:SIO_010079 ?oma_gene1 . # sio:SIO_010079 (is encoded by)
        ?oma_gene1 lscr:xrefEnsemblGene ?ensembl1 .
        ?oma_gene1 lscr:xrefNCBIGene ?ncbi1 .
        ?protein2 sio:SIO_010079 ?oma_gene2 .
        ?protein2 orth:organism ?oma_organism2 .
        ?oma_organism2 obo:RO_0002162 taxon:9606 .
        ?oma_gene2 lscr:xrefNCBIGene ?ncbi2 .
        ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2 .
      }
      GRAPH <http://metadb.riken.jp/db/uniprot_ncbigene> {
        ?uniprot2 rdfs:seeAlso ?ncbi2 .
        ?uniprot2 rdfs:seeAlso ?identifiers_ncbi2 .
        FILTER (?ncbi2 != ?identifiers_ncbi2)
      }
      GRAPH <http://metadb.riken.jp/db/gda_score_05> {
        ?gda sio:SIO_000628 ?identifiers_ncbi2 .
        ?gda sio:SIO_000628 ?umls .
        VALUES (?umls) { (umls:C0002395) } # Alzheimer disease
        FILTER (?identifiers_ncbi2 != ?umls)
      }
    }
  }
}
```

## Centralised query
**Expected runtime: 26 seconds**

[SPARQL query file](centralised_query.rq) 
```
PREFIX brso: <http://purl.jp/bio/10/brso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX lscr: <http://purl.org/lscr#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX genex: <http://purl.org/genex#>
PREFIX umls: <http://linkedlifedata.com/resource/umls/id/>

SELECT DISTINCT ?mouse ?homepage_mouse ?ensembl2
WHERE {
  GRAPH <http://metadb.riken.jp/db/bgee> {
    ?oma_gene2 a orth:Gene .
    ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2 .
    ?oma_gene2 orth:organism/obo:RO_0002162 taxon:9606 . # human
    ?oma_gene2 genex:isExpressedIn ?cond .
    ?cond genex:hasAnatomicalEntity obo:UBERON_0000451 . # prefrontal cortex
    ?expr genex:hasExpressionCondition ?cond .
    ?expr genex:hasSequenceUnit ?oma_gene2 .
    ?expr a genex:Expression .
    ?expr genex:hasConfidenceLevel obo:CIO_0000029 . # high confidence level
    ?expr genex:hasExpressionLevel ?exprLevel .
    FILTER (?exprLevel > 99)
  }
  {
    SELECT DISTINCT ?mouse ?homepage_mouse ?ensembl2
    WHERE {
      GRAPH <http://metadb.riken.jp/db/xsearch_animal_brso> {
        ?mouse brso:genomic_feature/brso:has_genomic_segment/rdfs:seeAlso ?mgi1 .
        ?mouse foaf:homepage ?homepage_mouse .
      }
      GRAPH <http://metadb.riken.jp/db/mgi_ncbi_ensembl> {
        ?mgi1 rdfs:seeAlso ?ensembl1 .
      }
      GRAPH <http://metadb.riken.jp/db/omaRDF> {
        ?cluster a orth:OrthologsCluster .
        ?cluster orth:hasHomologousMember ?node1 .
        ?cluster orth:hasHomologousMember ?node2 .
        FILTER (?node1 != ?node2)
        ?node1 orth:hasHomologousMember* ?protein1 .
        ?node2 orth:hasHomologousMember* ?protein2 .
        ?protein1 sio:SIO_010079 ?oma_gene1 . # sio:SIO_010079 (is encoded by)
        ?oma_gene1 lscr:xrefEnsemblGene ?ensembl1 .
        ?oma_gene1 lscr:xrefNCBIGene ?ncbi1 .
        ?protein2 sio:SIO_010079 ?oma_gene2 .
        ?protein2 orth:organism ?oma_organism2 .
        ?oma_organism2 obo:RO_0002162 taxon:9606 .
        ?oma_gene2 lscr:xrefNCBIGene ?ncbi2 .
        ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2 .
      }
      GRAPH <http://metadb.riken.jp/db/uniprot_ncbigene> {
        ?uniprot2 rdfs:seeAlso ?ncbi2 .
        ?uniprot2 rdfs:seeAlso ?identifiers_ncbi2 .
        FILTER (?ncbi2 != ?identifiers_ncbi2)
      }
      GRAPH <http://metadb.riken.jp/db/gda_score_05> {
        ?gda sio:SIO_000628 ?identifiers_ncbi2 .
        ?gda sio:SIO_000628 ?umls .
        VALUES (?umls) { (umls:C0002395) } # Alzheimer disease
        FILTER (?identifiers_ncbi2 != ?umls)
      }
    }
  }
}
```


## License
This repository is under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).
