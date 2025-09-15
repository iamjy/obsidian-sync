
```base
formulas:
  무제: ""
views:
  - type: table
    name: 표
    order:
      - file.name
      - tags
    sort:
      - property: file.name
        direction: ASC
  - type: cards
    name: 뷰
    cardSize: 100
    image: file.file
    imageFit: contain
  - type: cards
    name: 뷰 2
    image: file.file

```