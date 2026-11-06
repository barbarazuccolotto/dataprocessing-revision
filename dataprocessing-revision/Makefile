# Pipeline de processamento dos resultados de busca
#
#   Etapa 1 (format):  data/raw/<base>.<ext>  ->  data/01_formatted/input_<base>.xlsx
#   Etapa 2 (dedup):   data/01_formatted/*.xlsx  ->  data/02_deduplicated/summary_screened.xlsx
#
# Coloque os arquivos exportados em data/raw/, nomeados pelo tipo da base:
#   scopus.csv, wos.csv, pubmed.csv, pmc.txt, dimensions.csv, scholar.csv,
#   cochrane.csv, embase.csv, sciencedirect.ris, ieee.csv, bvs.csv, cab.csv,
#   scielo.csv, txt.txt
#
# Uso:
#   make install   # instala as dependências (requirements.txt)
#   make           # roda as duas etapas
#   make format    # roda só a etapa 1
#   make dedup     # roda a etapa 2 (e a 1, se necessário)
#   make clean     # apaga os resultados das etapas (mantém data/raw)

PYTHON     ?= python3
DATA_DIR   := data
RAW_DIR    := $(DATA_DIR)/raw
FORMAT_DIR := $(DATA_DIR)/01_formatted
DEDUP_DIR  := $(DATA_DIR)/02_deduplicated

FORMAT_SCRIPT := scripts/format_input.py
DEDUP_SCRIPT  := scripts/remove_duplicates.py

VALID_TYPES := scopus wos pubmed pmc dimensions scholar cochrane embase \
               sciencedirect ieee bvs cab scielo txt

RAW_FILES := $(wildcard $(RAW_DIR)/*.csv $(RAW_DIR)/*.txt $(RAW_DIR)/*.ris)
TYPES     := $(basename $(notdir $(RAW_FILES)))
INVALID   := $(filter-out $(VALID_TYPES),$(TYPES))
FORMATTED := $(patsubst %,$(FORMAT_DIR)/input_%.xlsx,$(TYPES))
DEDUPED   := $(DEDUP_DIR)/summary_screened.xlsx

empty :=
space := $(empty) $(empty)
comma := ,

.PHONY: all format dedup install check clean help

all: dedup

help:
	@sed -n '1,17p' Makefile

install:
	$(PYTHON) -m pip install -r requirements.txt

check:
ifeq ($(strip $(RAW_FILES)),)
	$(error Nenhum arquivo em $(RAW_DIR)/. Coloque ali os arquivos exportados (ex.: scopus.csv, wos.csv))
endif
ifneq ($(strip $(INVALID)),)
	$(error Nome(s) de arquivo inválido(s) em $(RAW_DIR)/: $(INVALID). Use um destes: $(VALID_TYPES))
endif
	@echo "Arquivos de entrada: $(notdir $(RAW_FILES))"

$(FORMAT_DIR) $(DEDUP_DIR):
	mkdir -p $@

# Etapa 1: uma regra por arquivo bruto (o tipo vem do nome do arquivo)
define FORMAT_RULE
$(FORMAT_DIR)/input_$(basename $(notdir $(1))).xlsx: $(1) $(FORMAT_SCRIPT) | check $(FORMAT_DIR)
	$(PYTHON) $(FORMAT_SCRIPT) -t $(basename $(notdir $(1))) -i $(1) -o $(FORMAT_DIR)
	@test -f $$@ || { echo "Erro: $$@ não foi gerado. Veja o log em $(FORMAT_DIR)/"; exit 1; }
endef
$(foreach f,$(RAW_FILES),$(eval $(call FORMAT_RULE,$(f))))

format: check $(FORMATTED)

# Etapa 2: junta todas as bases formatadas e remove duplicatas
$(DEDUPED): $(FORMATTED) $(DEDUP_SCRIPT) | $(DEDUP_DIR)
	@if [ $(words $(FORMATTED)) -lt 2 ]; then \
		echo "Erro: a remoção de duplicatas exige arquivos de pelo menos 2 bases diferentes."; exit 1; fi
	$(PYTHON) $(DEDUP_SCRIPT) -f $(subst $(space),$(comma),$(FORMATTED)) -o $(DEDUP_DIR)
	@test -f $@ || { echo "Erro: $@ não foi gerado. Veja o log em $(DEDUP_DIR)/"; exit 1; }

dedup: check $(DEDUPED)

clean:
	rm -rf $(FORMAT_DIR) $(DEDUP_DIR)
