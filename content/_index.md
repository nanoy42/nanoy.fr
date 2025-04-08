---
# Leave the homepage title empty to use the site title
title: ""
date: 2025-01-25
type: landing

design:
  # Default section spacing
  spacing: "6rem"

sections:
  - block: resume-biography-3
    content:
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: nanoy
      text: ""
      # Show a call-to-action button under your biography? (optional)
      button:
        text: Download CV
        url: uploads/resume.pdf
    design:
      css_class: black
      background:
        color: black
        image:
          # Add your image background to `assets/media/`.
          filename: stacked-peaks.svg
          filters:
            brightness: 1.0
          size: cover
          position: center
          parallax: true
  - block: markdown
    content:
      title: '📚 My Research'
      subtitle: ''
      text: |-

        My research falls in the general scope of Quantum Technologies, and more precisely on Quantum Communication. Click on the items below to discover more !

        {{< spoiler text="Development of Quantum Communication Protocols" >}}
          In my current research, I am still working on experimentally developing quantum communication protocols, such as Discrete-Variable and Continuous-Variable Quantum Key Distribution protocols or Quantum Random Number Generators. In particular, one of my goal is to increase their practicality and deployability. This research includes the development of open source software and transverse techniques.
        {{< /spoiler >}}

        {{< spoiler text="Integration of components for Quantum Communication" >}}
          The field of photonics and photonic-based quantum communication may greatly benefit from the monolithic integration of components, as it did for the field of electronics and computing. This however requires the development of advanced components with high-performance in order to reach the specific requirements of quantum protocols. It also require the development of specific techniques, but they can also have advantages such as size and stability. I also believe that hybrid integration will be required to reach complete integrated systems.
        {{< /spoiler >}}

        {{< spoiler text="Quantum Communication Infrastructures" >}}
          While many quantum communication protocols have been demonstrated in laboratory contexts, but it also important to demonstrate those protocols on deployed networks and fibers. Such deployments also require additional transverse techniques for synchronisation and impairments correction. It is also an opportunity to investigate the practical security of Quantum Key Distribution and how it can be combined with other protocols such as Post-Quantum Encryption to improve this practical security.
        {{< /spoiler >}}

        {{< spoiler text="Energetic Analysis of Quantum Communication Protocols" >}}
          With some of the earliest commercial devices arriving on the market, it is important to analyse their energetic cost before wider scale development. In addition, such a study could reveal another form of quantum advantage in the form of quantum protocols have fewer consumption than their classical counterpart. After an initial analysis of the cost of quantum key distribution protocols, I am now working on extending this analysis on the comparaison with classical protocols.
        {{< /spoiler >}}

        {{< spoiler text="Beyond Quantum Key Distribution" >}}
          Many techniques developed in the previous points can be used in protocols beyond Quantum Key Distribution. An example is the usage of balanced detectors, typically used in Continuous-Variable Quantum key Distribution, but that can also be used in hybrid protocols, entanglement based protocols or even quantum computation.
        {{< /spoiler >}}
    design:
      columns: '1'
  - block: cta-button-list
    content:
      # Need a custom icon?
      # Add an SVG image to the `assets/media/icons/` folder and reference it in the `icon` field below
      buttons:
        - text: Read my PhD thesis
          icon: hero/book-open
          url: https://thesis.nanoy.fr
    design:
      spacing:
        # Customize the section spacing. Order is top, right, bottom, left.
        padding: ['0', '0', '0', '0']
  - block: collection
    id: papers
    content:
      title: Featured Publications
      filters:
        folders:
          - publication
        featured_only: true
    design:
      view: article-grid
      columns: 2
  - block: collection
    content:
      title: Recent Publications
      text: ""
      filters:
        folders:
          - publication
        exclude_featured: false
    design:
      view: citation
  - block: collection
    id: talks
    content:
      title: Recent & Upcoming Talks
      filters:
        folders:
          - event
    design:
      view: article-grid
      columns: 1
  # - block: collection
  #   id: news
  #   content:
  #     title: Recent News
  #     subtitle: ''
  #     text: ''
  #     # Page type to display. E.g. post, talk, publication...
  #     page_type: post
  #     # Choose how many pages you would like to display (0 = all pages)
  #     count: 5
  #     # Filter on criteria
  #     filters:
  #       author: ""
  #       category: ""
  #       tag: ""
  #       exclude_featured: false
  #       exclude_future: false
  #       exclude_past: false
  #       publication_type: ""
  #     # Choose how many pages you would like to offset by
  #     offset: 0
  #     # Page order: descending (desc) or ascending (asc) date.
  #     order: desc
  #   design:
  #     # Choose a layout view
  #     view: date-title-summary
  #     # Reduce spacing
  #     spacing:
  #       padding: [0, 0, 0, 0]
  - block: cta-card
    demo: true # Only display this section in the Hugo Blox Builder demo site
    content:
      title: 👉 Build your own academic website like this
      text: |-
        This site is generated by Hugo Blox Builder - the FREE, Hugo-based open source website builder trusted by 250,000+ academics like you.

        <a class="github-button" href="https://github.com/HugoBlox/hugo-blox-builder" data-color-scheme="no-preference: light; light: light; dark: dark;" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="Star HugoBlox/hugo-blox-builder on GitHub">Star</a>

        Easily build anything with blocks - no-code required!
        
        From landing pages, second brains, and courses to academic resumés, conferences, and tech blogs.
      button:
        text: Get Started
        url: https://hugoblox.com/templates/
    design:
      card:
        # Card background color (CSS class)
        css_class: "bg-primary-700"
        css_style: ""
---
