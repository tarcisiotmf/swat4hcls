# Data in use for Alzheimer disease study: combining gene expression, orthology, bioresource and disease datasets

The queries below can be executed at https://knowledge.brc.riken.jp/bioresource/sparql.

## Federated query
**Expected runtime: 10 minutes**

[SPARQL query file](federated_query.rq) 
```
PREFIX brso: <http://purl.jp/bio/10/brso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX lscr: <http://purl.org/lscr#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX genex: <http://purl.org/genex#>
PREFIX umls: <http://linkedlifedata.com/resource/umls/id/>
PREFIX oma: <http://omabrowser.org/ontology/oma#>
PREFIX bgee: <http://bgee.org/#>

SELECT distinct ?mouse ?homepage_mouse ?ensembl2
WHERE {
   SERVICE <https://bgee.org/sparql/>{
      ?oma_gene2 a orth:Gene .
      ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2.
      ?oma_gene2 orth:organism/obo:RO_0002162 taxon:9606 .
      ?oma_gene2 genex:isExpressedIn ?cond.
      ?cond genex:hasAnatomicalEntity obo:UBERON_0000451 . # prefrontal cortex
      ?expr genex:hasExpressionLevel ?exprLevel .
         FILTER (?exprLevel > 99)
      ?expr genex:hasExpressionCondition ?cond.
      ?expr genex:hasSequenceUnit ?oma_gene2.
      ?expr a genex:Expression .
      ?expr genex:hasConfidenceLevel obo:CIO_0000029 . # high confidence level
      }
         {
        SELECT distinct ?mouse ?homepage_mouse ?ensembl2 WHERE {
         graph <http://metadb.riken.jp/db/xsearch_animal_brso> {
            ?mouse brso:genomic_feature/brso:has_genomic_segment/rdfs:seeAlso ?mgi1.
            ?mouse foaf:homepage ?homepage_mouse.
            }
         graph <http://metadb.riken.jp/db/mgi_ncbi_ensembl> {
            ?mgi1 rdfs:seeAlso ?ensembl1. 
            }
         graph <http://metadb.riken.jp/db/omaRDF> {
            ?cluster a orth:OrthologsCluster.
            ?cluster orth:hasHomologousMember ?node1.
            ?cluster orth:hasHomologousMember ?node2.
            ?node1 orth:hasHomologousMember* ?protein1. 
            ?node2 orth:hasHomologousMember* ?protein2. filter(?node1 != ?node2)
            ?protein1 sio:SIO_010079 ?oma_gene1. # sio:SIO_010079 (is encoded by)
            ?oma_gene1 lscr:xrefEnsemblGene ?ensembl1.
            ?oma_gene1 lscr:xrefNCBIGene ?ncbi1.
            ?protein2 sio:SIO_010079 ?oma_gene2 .
            ?protein2 orth:organism ?oma_organism2 .
            ?oma_organism2 obo:RO_0002162 taxon:9606 .
            ?oma_gene2 lscr:xrefNCBIGene ?ncbi2.
            ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2.
            }
         graph <http://metadb.riken.jp/db/uniprot_ncbigene> {
            ?uniprot2 rdfs:seeAlso ?ncbi2. 
            ?uniprot2 rdfs:seeAlso ?identifiers_ncbi2.   
               FILTER(?ncbi2 != ?identifiers_ncbi2)
            }
         graph <http://metadb.riken.jp/db/gda_score_05> {
            ?gda sio:SIO_000628 ?identifiers_ncbi2.
            ?gda sio:SIO_000628 ?umls.
                 values(?umls){(umls:C0002395)} # ALZ
                FILTER(?identifiers_ncbi2 != ?umls)
            }
          } limit 100
         }

      }
```

## Centralised query
**Expected runtime: 40 seconds**

[SPARQL query file](centralised_query.rq) 
```
PREFIX brso: <http://purl.jp/bio/10/brso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX lscr: <http://purl.org/lscr#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX genex: <http://purl.org/genex#>
PREFIX umls: <http://linkedlifedata.com/resource/umls/id/>
PREFIX oma: <http://omabrowser.org/ontology/oma#>
PREFIX bgee: <http://bgee.org/#>

SELECT distinct ?mouse ?homepage_mouse ?ensembl2
WHERE {
   graph <http://metadb.riken.jp/db/bgee> {
      ?oma_gene2 a orth:Gene .
      ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2.
      ?oma_gene2 orth:organism bgee:ORGANISM_9606 . # human
      ?oma_gene2 genex:isExpressedIn ?cond.
      ?cond genex:hasAnatomicalEntity obo:UBERON_0000451 . # prefrontal cortex
      ?expr genex:hasExpressionLevel ?exprLevel .
         FILTER (?exprLevel > 99)
      ?expr genex:hasExpressionCondition ?cond.
      ?expr genex:hasSequenceUnit ?oma_gene2.
      ?expr a genex:Expression .
      ?expr genex:hasConfidenceLevel obo:CIO_0000029 . # high confidence level
      }
         {
        SELECT distinct ?mouse ?homepage_mouse ?ensembl2 WHERE {
         graph <http://metadb.riken.jp/db/xsearch_animal_brso> {
            ?mouse brso:genomic_feature/brso:has_genomic_segment/rdfs:seeAlso ?mgi1.
            ?mouse foaf:homepage ?homepage_mouse.
            }
         graph <http://metadb.riken.jp/db/mgi_ncbi_ensembl> {
            ?mgi1 rdfs:seeAlso ?ensembl1. 
            }
         graph <http://metadb.riken.jp/db/omaRDF> {
            ?cluster a orth:OrthologsCluster.
            ?cluster orth:hasHomologousMember ?node1.
            ?cluster orth:hasHomologousMember ?node2.
            ?node1 orth:hasHomologousMember* ?protein1. 
            ?node2 orth:hasHomologousMember* ?protein2. filter(?node1 != ?node2)
            ?protein1 sio:SIO_010079 ?oma_gene1. # sio:SIO_010079 (is encoded by)
            ?oma_gene1 lscr:xrefEnsemblGene ?ensembl1.
            ?oma_gene1 lscr:xrefNCBIGene ?ncbi1.
            ?protein2 sio:SIO_010079 ?oma_gene2 .
            ?protein2 orth:organism ?oma_organism2 .
            ?oma_organism2 obo:RO_0002162 taxon:9606 .
            ?oma_gene2 lscr:xrefNCBIGene ?ncbi2.
            ?oma_gene2 lscr:xrefEnsemblGene ?ensembl2.
            }
         graph <http://metadb.riken.jp/db/uniprot_ncbigene> {
            ?uniprot2 rdfs:seeAlso ?ncbi2. 
            ?uniprot2 rdfs:seeAlso ?identifiers_ncbi2.   
               FILTER(?ncbi2 != ?identifiers_ncbi2)
            }
         graph <http://metadb.riken.jp/db/gda_score_05> {
            ?gda sio:SIO_000628 ?identifiers_ncbi2.
            ?gda sio:SIO_000628 ?umls.
                 values(?umls){(umls:C0002395)} # ALZ
                FILTER(?identifiers_ncbi2 != ?umls)
            }
          } limit 100
         }
      }
```


## License
This repository is under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).
