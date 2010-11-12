require 'rake/clean'
require 'rdf_context'
require 'net/http'

CLEAN.include 'ontology.rdf'

desc "Translate SPDX ontology from N3 into RDF/XML"
file 'ontology.rdf' => ['SPDX-2.0.html', 'SPDX-3.0.html', 'SPDX-4.0.html'] do 
  graph = RdfContext::Graph.new

  graph.merge! File.open('SPDX-2.0.html'){|f| RdfContext::RdfaParser.new.parse(f, 'http://spdx.org/spec')}
  graph.merge! File.open('SPDX-3.0.html'){|f| RdfContext::RdfaParser.new.parse(f, 'http://spdx.org/spec')}
  graph.merge! File.open('SPDX-4.0.html'){|f| RdfContext::RdfaParser.new.parse(f, 'http://spdx.org/spec')}

  File.open('ontology.rdf', 'w'){|f| graph.serialize(:format => :rdfxml, :io => f)}
end


desc "Abstract ontology produced by WonderWeb"
task "abstract" => 'ontology.rdf' do
  res = Net::HTTP.post_form(URI.parse('http://www.mygrid.org.uk/OWL/Validator'),
                            {'rdf' => File.read('ontology.rdf'), 'abstract' => 'yes', 'level' => 'Full'})
  doc = Nokogiri::HTML.parse(res.body)
  puts doc.at('div.box pre').content
end
