require 'rake/clean'
require 'rdf_context'
require 'net/http'
require 'mustache'

CLEAN.include 'ontology.rdf'

desc "Translate SPDX ontology from N3 into RDF/XML"
file 'spdx-1.0-ont.rdf' => 'spdx-1.0.html' do |t|
  graph = File.open('spdx-1.0.html'){|f| RdfContext::RdfaParser.new.parse(f, 'http://spdx.org/spec')}

  File.open(t.name, 'w'){|f| graph.serialize(:format => :rdfxml, :io => f)}
end


desc "Abstract ontology produced by WonderWeb"
task "abstract" => 'spdx-1.0-ont.rdf' do
  res = Net::HTTP.post_form(URI.parse('http://www.mygrid.org.uk/OWL/Validator'),
                            {'rdf' => File.read('spdx-1.0-ont.rdf'), 'abstract' => 'yes', 'level' => 'Full'})
  doc = Nokogiri::HTML.parse(res.body)
  elem = doc.at('div.box pre')

  if elem.nil?
    STDERR.puts doc.at('//body').to_s
    raise "OWL validation seems to have failed!"
  end

  puts elem.content
end

SPDX_SPEC_FILE_NAME = 'spdx-1.0.html'
desc "Compile complete spec from sections"
file SPDX_SPEC_FILE_NAME => (FileList['*.html'] - FileList[SPDX_SPEC_FILE_NAME]) do |t|

  compiler = Class.new(Mustache) do 
    self.template_path = File.dirname(__FILE__)
    self.template_extension = 'html'
    self.template_file = './spec.html'
  end

  File.open(t.name, 'w'){|f| f.write compiler.render}
end

