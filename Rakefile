require 'rake/clean'
require 'rdf_context'

CLEAN.include 'ontology.rdf'

desc "Translate SPDX ontology from N3 into RDF/XML"
file 'ontology.rdf' => 'ontology.n3' do 
  graph = File.open('SPDX-2.0.html'){|f| RdfContext::RdfaParser.new.parse(f, 'http://spdx.org/spec')}
  File.open('ontology.rdf', 'w'){|f| graph.serialize(:format => :rdfxml, :io => f)}
end

