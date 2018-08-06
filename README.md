# TiddlyWiki on SiT

This project is about using https://github.com/sit-fyi/sit as a backend for TiddlyWiki.

# Rationale

There at least two aspects where alternative backend storage is useful.

## Project scratch-pad which travels with code repository

We firmly believe that all software development artifacts must travel with the code.
Use TiddlyWiki for storing thought process about how and why certain feature were
developed makes total sense. However storing HTML file in the repository is not
practical since it makes PR reviews harder due to HTML noise.

TiddlyWiki has server implementation which is using `nodejs`. It is file based
storage which stores every tiddler in separate file. However it is quite
complicated and awkward to use. There are following problems with `nodejs`
implementation:

- manual addition of tiddler files into tiddlers directory wouldn't make
  them available in TiddlyWiki

## TiddlyWiki on smartphones

`nodejs` based installation of TiddliWiki on a smartphone is very involved process.
Alternative self contained backend might help here.
