OBO=http://purl.obolibrary.org/obo
ONT=peco
BASE=$(OBO)/$(ONT)
#SRC=plant-trait-ontology.obo.owl
SRC=peco-edit.obo
RELEASEDIR=../..
ROBOT= robot
OWLTOOLS= owltools


all: all_imports all_patterns $(ONT).owl $(ONT).obo
test: all
prepare_release: all
	cp $(ONT).owl $(ONT).obo $(RELEASEDIR) &&\
	mkdir -p $(RELEASEDIR)/imports &&\
	cp imports/*{owl,obo} $(RELEASEDIR)/imports &&\
	git add $(RELEASEDIR)/imports/*{obo,owl} &&\
	(test -f subsets/*owl && cp subsets/*{owl,obo} $(RELEASEDIR)/subsets && git add $(RELEASEDIR)/subsets/* || echo "no subsets") &&\
	echo "Release files are now in $(RELEASEDIR) - now you should commit, push and make a release on github"

$(SRC).owl: $(SRC)
	$(ROBOT) convert -i $< -o $@

$(ONT).owl: $(SRC)
	$(ROBOT)  reason -i $< -r ELK relax reduce -r ELK annotate -V $(BASE)/releases/`date +%Y-%m-%d`/$(ONT).owl -o $@
$(ONT).obo: $(ONT).owl
	$(ROBOT) convert -i $< -f obo --check false -o $(ONT).obo.tmp && grep -v '^owl-axioms:' $(ONT).obo.tmp > $@ && rm $(ONT).obo.tmp


IMPORTS = chebi ncbitaxon ro envo
IMPORTS_OWL = $(patsubst %, imports/%_import.owl,$(IMPORTS)) $(patsubst %, imports/%_import.obo,$(IMPORTS))

# Make this target to regenerate ALL
all_imports: $(IMPORTS_OWL)

# Use ROBOT, driven entirely by terms lists NOT from source ontology
imports/%_import.owl: mirror/%.owl imports/%_terms.txt
	$(ROBOT) extract -i $< -T imports/$*_terms.txt --method BOT -O  $(BASE)/$@ -o $@
.PRECIOUS: imports/%_import.owl

imports/%_import.obo: imports/%_import.owl
	$(ROBOT) convert -i $< -f obo --check false -o $@

# clone remote ontology locally, perfoming some excision of relations and annotations
mirror/%.obo: $(SRC) 
	test -d mirror || mkdir mirror &&\
	wget --no-check-certificate $(OBO)/$*.obo -O $@ && touch $@
.PRECIOUS: mirror/%.obo
mirror/%.owl: mirror/%.obo
	$(OWLTOOLS) --no-debug $< --remove-annotation-assertions -l -s -d --remove-dangling-annotations --set-ontology-id $(OBO)/$*.owl  -o $@
.PRECIOUS: mirror/%.owl

release: $(ONT).owl $(ONT).obo
	cp $^ $(RELEASEDIR)

#######PATTERNS
PATTERNS_OWL = $(patsubst %.tsv, %_pattern.owl, $(wildcard patterns/*.tsv)) $(patsubst %.tsv, %_pattern.obo, $(wildcard patterns/*.tsv))

all_patterns: $(PATTERNS_OWL)

patterns/%_pattern.owl: patterns/%.tsv
	dosdp-tools --outfile=$@ --prefixes=patterns/curie_map.yaml --obo-prefixes=true --template=patterns/peco$(findstring _role,$*).yaml --ontology=imports/$(patsubst exposure-%,%, $(patsubst %_role,%,$*))_import.owl generate --infile=patterns/$*.tsv
	robot annotate -i $@ -o $@ -O $(BASE)/$@
	#patterns/apply-pattern.py -P patterns/curie_map.yaml -i patterns/$*.tsv -p patterns/eo.yaml -n $@ > $@
patterns/%_pattern.obo: patterns/%_pattern.owl
	$(OWLTOOLS) $< -o -f obo $@


#print var function for test
print-%:
	@echo $* = $($*)