Visualização de Dados de Microbioma

Este script gera três tipos de visualizações a partir de dados de sequenciamento processados com QIIME 2:

Barplot de abundância relativa por filo;

Diversidade alfa (Índice de Shannon) por nível de interferência antrópica;

Heatmap da abundância taxonômica por gênero.

Dependências
library(tidyverse)
library(qiime2R)

Diretório de trabalho
setwd("/media/yas/HD_YASMIN1/LAFISBIO/R")

1. Barplot de Abundância Relativa
1.1 Importação dos dados
metadata <- read_tsv(
  "2_1_manifest.tsv",
  show_col_types = FALSE
) %>%
  rename(SampleID = 1)

SVs_raw <- read_qza("table.qza")$data

taxonomy <- read_qza("taxonomy.qza")$data %>%
  parse_taxonomy()

1.2 Transformação para abundância relativa

As contagens de cada amostra são normalizadas para porcentagem, permitindo a comparação entre bibliotecas de sequenciamento com diferentes profundidades.

SVs_rel <- apply(
  SVs_raw,
  2,
  function(x) (x / sum(x)) * 100
)

1.3 Agrupamento taxonômico

Neste exemplo, os dados são agrupados ao nível de Filo (Phylum).

Para utilizar outros níveis taxonômicos, substitua $Phylum por:

$Class — Classe

$Order — Ordem

$Family — Família

$Genus — Gênero

taxasums_rel <- summarize_taxa(
  SVs_rel,
  taxonomy
)$Phylum

1.4 Construção do gráfico
plot_bar <- taxa_barplot(
  taxasums_rel,
  metadata,
  "human_interference"
) +
  ylab("Abundância Relativa (%)") +
  theme_q2r() +
  theme(
    axis.text.x = element_text(
      angle = 45,
      hjust = 1,
      size = 10,
      face = "bold"
    ),
    legend.position = "right",
    legend.text = element_text(size = 8),
    legend.title = element_text(
      face = "bold",
      size = 10
    )
  )

1.5 Exportação
ggsave(
  "barplot_abundancia_relativa_fungos.pdf",
  plot = plot_bar,
  height = 6,
  width = 12,
  device = "pdf"
)

2. Diversidade Alfa — Shannon
2.1 Importação dos metadados
metadata <- read_tsv(
  "2_1_manifest.tsv",
  show_col_types = FALSE
) %>%
  rename(SampleID = 1)

2.2 Importação dos valores de Shannon

O vetor de diversidade de Shannon previamente calculado no QIIME 2 é importado a partir do arquivo shannon_vector.qza.

shannon <- read_qza(
  "shannon_vector.qza"
)$data %>%
  rownames_to_column("SampleID")

2.3 Integração com os metadados
df_shannon <- metadata %>%
  left_join(
    shannon,
    by = "SampleID"
  ) %>%
  filter(
    !is.na(shannon_entropy)
  )

2.4 Construção do gráfico

O gráfico apresenta a distribuição do índice de Shannon entre os diferentes níveis de interferência antrópica.

plot_alpha <- df_shannon %>%
  ggplot(
    aes(
      x = human_interference,
      y = shannon_entropy,
      fill = human_interference
    )
  ) +
  geom_boxplot(
    alpha = 0.6,
    outlier.shape = NA
  ) +
  geom_jitter(
    shape = 21,
    width = 0.2,
    height = 0,
    color = "black",
    size = 3
  ) +
  labs(
    x = "Nível de Interferência Antrópica",
    y = "Diversidade Alfa (Índice de Shannon)"
  ) +
  theme_q2r() +
  scale_fill_viridis_d(
    option = "plasma"
  ) +
  theme(
    legend.position = "none",
    axis.text.x = element_text(
      size = 11,
      face = "bold"
    ),
    axis.title = element_text(
      size = 12,
      face = "bold"
    )
  )

2.5 Exportação
ggsave(
  "Shannon_por_Interferencia.pdf",
  plot = plot_alpha,
  height = 5,
  width = 6,
  device = "pdf"
)

3. Heatmap Taxonômico
3.1 Importação dos dados
metadata <- read_tsv(
  "2_1_manifest.tsv",
  show_col_types = FALSE
) %>%
  rename(SampleID = 1)

SVs <- read_qza(
  "table.qza"
)$data

taxonomy <- read_qza(
  "taxonomy.qza"
)$data %>%
  parse_taxonomy()

3.2 Agrupamento por gênero

Neste exemplo, os dados são agrupados ao nível de Gênero.

taxasums <- summarize_taxa(
  SVs,
  taxonomy
)$Genus

3.3 Construção do heatmap
plot_heatmap <- taxa_heatmap(
  taxasums,
  metadata,
  "human_interference"
)

3.4 Exportação
ggsave(
  "heatmap.pdf",
  plot = plot_heatmap,
  height = 4,
  width = 8,
  device = "pdf"
)

Arquivos de entrada

O script espera encontrar os seguintes arquivos no diretório de trabalho:

2_1_manifest.tsv
table.qza
taxonomy.qza
shannon_vector.qza

Arquivos de saída

Após a execução, serão gerados:

barplot_abundancia_relativa_fungos.pdf
Shannon_por_Interferencia.pdf
heatmap.pdf

Observação

O caminho definido em setwd() corresponde a um diretório local específico. Para compartilhar o projeto no GitHub, recomenda-se substituir esse caminho absoluto por uma estrutura de projeto relativa ou utilizar um projeto do RStudio (.Rproj).# Qiime2R

# SCRIPT: Barplot ==============================================================

library(tidyverse)
library(qiime2R)

# 1. Definição do Diretório de Trabalho
setwd("/media/yas/HD_YASMIN1/LAFISBIO/R")

metadata <- read_tsv("2_1_manifest.tsv") %>%
  rename(SampleID = 1)
  
SVs_raw <- read_qza("table.qza")$data
taxonomy <- read_qza("taxonomy.qza")$data %>% parse_taxonomy()

# 3. Transformação para Abundância Relativa (%)
# Normaliza as bibliotecas de sequenciamento desiguais
SVs_rel <- apply(SVs_raw, 2, function(x) (x / sum(x)) * 100)

# 4. Agrupamento Taxonômico
# Para detalhar mais, alterar o  $Phylum para $Class, $Order, $Family ou $Genus.
taxasums_rel <- summarize_taxa(SVs_rel, taxonomy)$Phylum


plot_bar <- taxa_barplot(taxasums_rel, metadata, "human_interference") +
  ylab("Abundância Relativa (%)") +
  theme_q2r() +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1, size = 10, face = "bold"),
    legend.position = "right",
    legend.text = element_text(size = 8),
    legend.title = element_text(face = "bold", size = 10)
  )

ggsave("barplot_abundancia_relativa_fungos.pdf", plot = plot_bar, height = 6, width = 12, device = "pdf")

# =====================================================================
# SCRIPT: Diversidade Alfa (Shannon) 

library(tidyverse)
library(qiime2R)

setwd("/media/yas/HD_YASMIN1/LAFISBIO/R")


metadata <- read_tsv("2_1_manifest.tsv", show_col_types = FALSE) %>%
  rename(SampleID = 1)


shannon <- read_qza("shannon_vector.qza")$data %>% 
  rownames_to_column("SampleID")

# 2. Integração dos metadados com os valores de diversidade
df_shannon <- metadata %>%
  left_join(shannon, by="SampleID") %>%
  filter(!is.na(shannon_entropy)) 


plot_alpha <- df_shannon %>%
  ggplot(aes(x = human_interference, y = shannon_entropy, fill = human_interference)) +
  geom_boxplot(alpha = 0.6, outlier.shape = NA) + 
  geom_jitter(shape = 21, width = 0.2, height = 0, color = "black", size = 3) +
  labs(
    x = "Nível de Interferência Antrópica", 
    y = "Diversidade Alfa (Índice de Shannon)"
  ) +
  theme_q2r() +
  scale_fill_viridis_d(option = "plasma") + 
  theme(legend.position = "none",
        axis.text.x = element_text(size = 11, face = "bold"),
        axis.title = element_text(size = 12, face = "bold"))

ggsave("Shannon_por_Interferencia.pdf", plot = plot_alpha, height = 5, width = 6, device = "pdf")

# =====================================================================
# SCRIPT: Heatmap Taxonômico (

library(tidyverse)
library(qiime2R)

setwd("/media/yas/HD_YASMIN1/LAFISBIO/R")

metadata <- read_tsv("2_1_manifest.tsv", show_col_types = FALSE) %>%
  rename(SampleID = 1)

SVs <- read_qza("table.qza")$data
taxonomy <- read_qza("taxonomy.qza")$data %>% parse_taxonomy()

taxasums <- summarize_taxa(SVs, taxonomy)$Genus

taxa_heatmap(taxasums, metadata, "human_interference")

ggsave("heatmap.pdf", height = 4, width = 8, device = "pdf")


