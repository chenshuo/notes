# Notes on Git

## Data model

* `blob` is content of file
```text
$ git cat-file -p 74576250e3c3a901bf06832bd0e4ee5e00422115
# Shuo's notes
http://chenshuo.github.io/notes
```

* `tree` has one or more `tree`/`blob`, it's [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree).
    - `blob` are leaf nodes
    - `tree` are internal nodes
```text
$ git cat-file -p 5d42664e3d812288b47795718d0e4faf26397bf9
100644 blob cde88fe0e10f265ed3cb2af0ea5422f607c1a37c    .gitignore
100644 blob 74576250e3c3a901bf06832bd0e4ee5e00422115    README.md
040000 tree eabff4ee0b30f5d6a644ce508ba7fe47dc563aa4    docs
100644 blob 020682e2399c75cf3ad9636964c7501f27a0e893    mkdocs.yml
```

* `commit`:
    -  has one `tree`
    -  has one or more parent `commit`(s)
```text
$ git cat-file -p 3f08891b366c750d4739241874cfb1dbef56c398
tree 5d42664e3d812288b47795718d0e4faf26397bf9
parent 955f683cae5d2514d1c788fce9206451e2ac3334
author Shuo Chen <chenshuo@chenshuo.com> 1484903215 -0800
committer Shuo Chen <chenshuo@chenshuo.com> 1484903215 -0800

update ssl.md
```

* `tag` ???

* Exceptions:
    - initial `commit` has no parent
    - empty file is empty `blob`
    - empty directory is empty `tree`

## Pack file format

### Delta

## Pack index format
