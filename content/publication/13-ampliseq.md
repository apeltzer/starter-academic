+++
title = "Interpretations of microbial community studies are biased by the selected 16S rRNA gene amplicon sequencing pipeline."
date = 2019-12-19T00:00:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Straub D", "Blackwell N", "Langarica Fuentes A", "Peltzer A", "Nahnsen S", "Kleindienst S"]

# Publication type.
# Legend:
# 0 = Uncategorized
# 1 = Conference paper
# 2 = Journal article
# 3 = Manuscript
# 4 = Report
# 5 = Book
# 6 = Book section
publication_types = ["2"]

# Publication name and optional abbreviated version.
publication = "In *BiorXiV*"
publication_short = "In *BiorXiV*"

# Abstract and optional shortened version.
abstract = "One of the major methods to identify microbial community composition, to unravel microbial population dynamics, and to explore microbial diversity in environmental samples is DNA- or RNA-based 16S rRNA (gene) amplicon sequencing. Subsequent bioinformatics analyses are required to extract valuable information from the high-throughput sequencing approach. However, manifold bioinformatics tools complicate their choice and might cause differences in data interpretation, making the selection of the pipeline a crucial step. Here, we compared the performance of most widely used 16S rRNA gene amplicon sequencing analysis tools (i.e. Mothur, QIIME1, QIIME2, and MEGAN) using mock datasets and environmental samples from contrasting terrestrial and freshwater sites. Our results showed that QIIME2 outcompeted all other investigated tools in sequence recovery (>10 times less false positives), taxonomic assignments (>22% better F-score) and diversity estimates (>5% better assessment), while there was still room for improvement e.g. imperfect sequence recovery (recall up to 87%) or detection of additional false sequences (precision up to 72%). Furthermore, we found that microbial diversity estimates and highest abundant taxa varied among analysis pipelines (i.e. only one in five genera was shared among all analysis tools) when analyzing environmental samples, which might skew biological conclusions. Our findings were subsequently implemented in a high-performance computing conformant workflow following the FAIR (Findable, Accessible, Interoperable, and Re-usable) principle, allowing reproducible 16S rRNA gene amplicon sequence analysis starting from raw sequence files. Our presented workflow can be utilized for future studies, thereby facilitating the analysis of high-throughput DNA- or RNA-based 16S rRNA (gene) sequencing data substantially."

# Featured image thumbnail (optional)
image_preview = ""

# Is this a selected publication? (true/false)
selected = false

# Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's filename without extension.
#   E.g. `projects = ["deep-learning"]` references `content/project/deep-learning.md`.
#   Otherwise, set `projects = []`.
projects = []

# Tags (optional).
#   Set `tags = []` for no tags, or use the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["amplicon", "16S", "rRNA", "amplicon sequencing", "environmental omics"]

# Links (optional).
#url_pdf = "#"
#url_preprint = "#"
#url_code = "#"
#url_dataset = "#"
#url_project = "#"
#url_slides = "#"
#url_video = "#"
#url_poster = "#"
url_source = "https://www.biorxiv.org/content/10.1101/2019.12.17.880468v1"

# Custom links (optional).
#   Uncomment line below to enable. For multiple links, use the form `[{...}, {...}, {...}]`.
#url_custom = [{name = "Custom Link", url = "http://example.org"}]

# Does this page contain LaTeX math? (true/false)
math = false

# Does this page require source code highlighting? (true/false)
highlight = false

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
#[header]
#image = "headers/bubbles-wide.jpg"
#caption = "My caption :smile:"

+++
