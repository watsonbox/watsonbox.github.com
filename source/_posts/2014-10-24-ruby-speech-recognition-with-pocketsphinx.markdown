---
layout: post
title: "Ruby speech recognition with Pocketsphinx"
date: 2014-10-24 15:42:02 +0200
comments: true
categories: [Ruby, Speech Recognition]
---

[pocketsphinx-ruby](https://github.com/watsonbox/pocketsphinx-ruby) is a high-level Ruby wrapper for the pocketsphinx C API. It uses the Ruby Foreign Function Interface (FFI) to directly load and call functions in libpocketsphinx, as well as libsphinxav for recoding live audio using a number of different audio backends.

The goal of the project is to make it as easy as possible for the Ruby community to experiment with speech recognition, in particular for use in grammar-based command and control applications. Setting up a real time recognizer is as simple as:

```ruby
configuration = Pocketsphinx::Configuration::Grammar.new do
  sentence "Go forward ten meters"
  sentence "Go backward ten meters"
end

Pocketsphinx::LiveSpeechRecognizer.new(configuration).recognize do |speech|
  puts speech
end
```

This library supports Ruby MRI 1.9.3+, JRuby, and Rubinius. It depends on the current development versions of Pocketsphinx and Sphinxbase - there are [Homebrew recipes](https://github.com/watsonbox/homebrew-cmu-sphinx) available for a quick start on OSX.