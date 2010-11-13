require 'rake/clean'
require 'rdf_context'
require 'net/http'
require 'mustache'

CLEAN.include 'ontology.rdf'

desc "Translate SPDX ontology from N3 into RDF/XML"
file 'ontology.rdf' => 'spdx-complete-1.0.html' do 
  graph = File.open('spdx-complete-1.0.html'){|f| RdfContext::RdfaParser.new.parse(f, 'http://spdx.org/spec')}

  File.open('ontology.rdf', 'w'){|f| graph.serialize(:format => :rdfxml, :io => f)}
end


desc "Abstract ontology produced by WonderWeb"
task "abstract" => 'ontology.rdf' do
  res = Net::HTTP.post_form(URI.parse('http://www.mygrid.org.uk/OWL/Validator'),
                            {'rdf' => File.read('ontology.rdf'), 'abstract' => 'yes', 'level' => 'Full'})
  doc = Nokogiri::HTML.parse(res.body)
  elem = doc.at('div.box pre')

  if elem.nil?
    STDERR.puts doc.at('//body').to_s
    raise "OWL validation seems to have failed!"
  end

  puts elem
end


desc "Compile complete spec from sections"
file "spdx-complete-1.0.html" => (FileList['*.html'] - FileList['spdx-complete-1.0.html']) do |t|
  File.open("spdx-complete-1.0.html", 'w'){|f| f.write SpdxCompile.render}
end

class SpdxCompile < Mustache
  self.template_path = File.dirname(__FILE__)
  self.template_extension = 'html'
  self.template_file = './SPDX-1.0.html'
end
