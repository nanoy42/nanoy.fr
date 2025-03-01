---
title: "QOSST: A Highly-Modular Open Source Platform for Experimental Continuous-Variable Quantum Key Distribution"
authors:
- nanoy
- Matteo Schiavon
- Valentina Marulanda Acosta
- Baptiste Gouraud
- Luis Trigo Vidarte
- Philippe Grangier
- Amine Rhouni
- Eleni Diamanti
- 
date: "2024-12-23T00:00:00Z"
doi: "10.22331/q-2024-12-23-1575"

# Schedule page publish date (NOT publication's date).
publishDate: "2024-12-23T00:00:00Z"

# Publication type.
# Accepts a single type but formatted as a YAML list (for Hugo requirements).
# Enter a publication type from the CSL standard.
publication_types: ["article-journal"]

# Publication name and optional abbreviated publication name.
publication: "Quantum 8, 1575 (2024)."
publication_short: ""

abstract: Quantum Key Distribution (QKD) enables secret key exchange between two remote parties with information-theoretic security rooted in the laws of quantum physics. Encoding key information in continuous variables (CV), such as the values of quadrature components of coherent states of light, brings implementations much closer to standard optical communication systems, but this comes at the price of significant complexity in the digital signal processing techniques required for operation at low signal-to-noise ratios. In this work, we wish to lower the barriers to entry for CV-QKD experiments associated to this difficulty by providing a highly modular, open source software that is in principle hardware agnostic and can be used in multiple configurations. We benchmarked this software, called QOSST, using an experimental setup with a locally generated local oscillator, frequency multiplexed pilots and RF-heterodyne detection, and obtained state-of-the-art secret key rates of the order of Mbit/s over metropolitan distances at the asymptotic limit. We hope that QOSST can be used to stimulate further experimental advances in CV-QKD and be improved and extended by the community to achieve high performance in a wide variety of configurations.

# Summary. An optional shortened abstract.
summary: 
tags:
  - CV-QKD

featured: true

# links:
# - name: ""
#   url: ""
url_pdf: https://quantum-journal.org/papers/q-2024-12-23-1575/pdf/
url_code: 'https://github.com/qosst'
url_dataset: ''
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: 'Logo of the QOSST software'
  focal_point: ""
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: 
---