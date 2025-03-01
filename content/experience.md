---
title: 'Experience'
date: 2023-10-24
type: landing

design:
  spacing: '5rem'

# Note: `username` refers to the user's folder name in `content/authors/`

# Page sections
sections:
  - block: markdown
    content:
      title: _Curriculum vitæ_
      text: ""
    design:
      spacing:
        # Customize the section spacing. Order is top, right, bottom, left.
        padding: ['100px', '0', '0px', '0']
  - block: cta-button-list
    content:
      buttons:
        - text: Download my CV
          icon: academicons/cv
          url: /uploads/resume.pdf
    design:
      spacing:
        # Customize the section spacing. Order is top, right, bottom, left.
        padding: ['0px', '0', '0px', '0']
  - block: resume-experience
    content:
      username: nanoy
    design:
      # Hugo date format
      date_format: 'January 2006'
      # Education or Experience section first?
      is_education_first: false
    design:
      spacing:
        # Customize the section spacing. Order is top, right, bottom, left.
        padding: ['20px', '0', '0px', '0']
  - block: resume-skills
    content:
      title: Skills & Hobbies
      username: nanoy
    design:
      show_skill_percentage: false
  # - block: resume-awards
  #   content:
  #     title: Awards
  #     username: nanoy
  - block: resume-languages
    content:
      title: Languages
      username: nanoy
---