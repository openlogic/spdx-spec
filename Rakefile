require 'rdf/n3'
require 'rdf/rdfxml'

desc "Translate SPDX ontology from N3 into RDF/XML"
file 'ontology.rdf' => 'ontology.n3' do 
  graph = RDF::Graph.new do |graph|
    RDF::N3::Reader.open('ontology.n3') do |reader|    
      reader.each_statement do |statement|
        graph << statement
      end
    end
  end

  RDF::RDFXML::Writer.open('ontology.rdf'){|writer| writer << graph}
end
