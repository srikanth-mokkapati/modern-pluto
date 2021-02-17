---
title: Contact Me
hide_title: false
sections:
  - section_id: contact-form
    type: section_form
    content: >
      To get in touch please fill the form below or mail me directly on
      srikanth.mokkapati9@gmail.com
    form_id: contactForm
    form_action: /thank-you
    form_fields:
      - input_type: text
        name: name
        label: Name
        default_value: You Name
        is_required: true
      - input_type: email
        name: email
        label: Email
        default_value: Your Email
        is_required: true
      - input_type: text
        name: subject
        label: Subject
        default_value: Subject
        options:
          - Error on the site
          - Sponsorship
          - Other
      - input_type: textarea
        name: message
        label: Message
        default_value: Your message
      - input_type: checkbox
        name: consent
        label: >-
          I understand that this form is storing my submitted information so I
          can be contacted.
    submit_label: Send Message
seo:
  title: Contact Me
  description: This is the contact page me page of Srikanth Mokkapati
  extra:
    - name: 'og:type'
      value: website
      keyName: property
    - name: 'og:title'
      value: Contact
      keyName: property
    - name: 'og:description'
      value: This is the contact me page of Srikanth Mokkapati
      keyName: property
    - name: 'twitter:card'
      value: summary
    - name: 'twitter:title'
      value: Contact
    - name: 'twitter:description'
      value: This is the contact me page of Srikanth Mokkapati
    - name: 'og:image'
      value: /images/DSCN2884.JPG
      keyName: property
      relativeUrl: true
    - name: 'twitter:image'
      value: /images/DSCN2884.JPG
      keyName: property
      relativeUrl: true
template: advanced
excerpt: Contact Me
---
