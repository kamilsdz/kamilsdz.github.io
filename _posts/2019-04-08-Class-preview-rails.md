---
layout: post
title: Quick class preview from rails console
---

One think I really like in Rubymine is that it knows where are classes are and thanks to that we can move to them with one click. After moving to VIM I installed [vim-definitive](https://vimawesome.com/plugin/vim-definitive) plugin that adds this future. Unfortunatelly, it doesn't work with ruby gems, we can go to definitions in project, but not to gems. 

But I have a pretty cool tip for you - thanks to `pry`, we can preview class from terminal simple using `show-source`

```
pry(main)> show-source ClosureTree

From: /Users/kamilsopata/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/closure_tree-7.0.0/lib/closure_tree.rb @ line 3:
Module name: ClosureTree
Number of lines: 23

module ClosureTree
  extend ActiveSupport::Autoload

  autoload :HasClosureTree
  autoload :HasClosureTreeRoot
  autoload :Support
  autoload :HierarchyMaintenance
  autoload :Model
  autoload :Finders
  autoload :HashTree
  autoload :Digraphs
  autoload :DeterministicOrdering
  autoload :NumericDeterministicOrdering
  autoload :Configuration

  def self.configure
    yield configuration
  end

  def self.configuration
    @configuration ||= Configuration.new
  end
end
```

We have the file path and the file content. Simple as that!
