
Getting started
=====

Invoking `rake` in the root directory of the project will build the following artifacts:

 * `build/spdx-DRAFT.html`        : The complete spec.
 * `build/spdx-ont-DRAFT.rdf`     : An RDF/XML representation of the RDF ontology.
 * `build/spdx-grammar-DRAFT.txt` : A formal specification of the tag-value syntax.
 
The files can also be generated individually by invoking `rake <file-name>`.

An abstract representation of the RDF ontology defined in the spec can be generated (via WonderWeb) by `rake abstract`.

Making changes:

 5. Create bug at
    <http://bugs.linux-foundation.org/enter_bug.cgi?product=spdx>
    describing the issue.
 2. Clone the repository.
 3. Create a feature branch.
 4. Make your changes on that branch.
 5. Publish your branch.
 6. Add a comment describing your changes with a link to your
    branch and attach an archive of the build directory.

To use the commands listed above you will need the follow steps.

 1. Install Ruby 1.9.2 using the appropriate mechanism for your OS.
    (Earlier versions of Ruby may work but have not been tested.)
 2. Install sqlite3-ruby gem: 
    `gem install --source http://rubygems.org sqlite3-ruby --version 1.2.5`
 4. Install various other gems: 
    `gem install --source http://rubygems.org rdf_context mustache git`

