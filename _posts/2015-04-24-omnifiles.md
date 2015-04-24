---
layout: post
title: omnifiles
---

Recently I wrote a small web application for storing temporary files and providing short links to them. It is like a shortener service integrated with file storage. I had found a few simple services for screenshots but it was needed to store pdfs and archives sometimes. Another requirement was to allow easy access by curl both for accessing (downloading) and for storing (uploading). And of course I wanted to store sensitive files at my own server.

This application called omnifiles is open-source and can be found at [https://github.com/theirix/omnifiles](https://github.com/theirix/omnifiles). omnifiles is a hobby project and a playground to tighten my web skills. I am using it for my own needs but you are welcome to provide feedback or patches!

Let's talk a little about it's architecture. It is a simple web app built with Ruby Sinatra framework, HAML for markup, MongoDB for metadata storage and filesystem for file storage.

## Storage

Files are stored in filesystem with their unique shortened names. MongoDB contains documents for each shortened link containing shortened link itself (acts as a key), original filename and MIME type, access statistics. Initially metadata was stored in sqlite. Certainly there is no need in scaling, sharding and other CAP stuff but hey, it is 2015! It is reasonable to use NoSQL if there is no strict need in structured data.

I used MongoDB 2.6 and stable ruby driver. Ruby driver was recently [rewritten to version 2.0](https://www.mongodb.com/blog/post/announcing-ruby-driver-20-rewrite) to support Mongo 3.0 but for now a stable version is sufficient and easy to use.

## API

omnifiles should be simple and easily accessible.
Requests to store files are sent using POST requests. POST request emits a shortened URL as a response body. I use a curl utility for command-line posting.

There are two ways to store a file (omitting auth and url).

1. Send a POST form with a single file field:

        % curl -F "file=@file.jpg" ...

2. Send a file using POST binary stream:

        % curl -H "Content-Type: application/octet-stream" --data-binary "@file.jpg" ...

Both variants are not very concise. I do not like an artificial form field at the first variant. Second variant just streams a file as a request body. Another vote against first variant is about intermediate form file saving to the temporary directory by the Rack middleware. You can be more efficient with a stream.

File is downloaded by GET request with a shortened link:

        % curl http://localhost/sge36a

Omnifiles provides an original filename as an additional header if a client wants to rename a downloaded file after downloading. Another nice feature is to return a saved MIME type to the response so browser can show images or pdfs directly inside browser window.

I added control panel to omnifiles to view a single file statistics (`http://localhost/stat/sge36a`), a whole storage statistics (`http://localhost/stat`) or to delete unneeded files.

## Web

Web frontend and backend are written in Sinatra. In omnifiles API and presentation are not clearly separated for the sake of simplicity. For example, routes are not classic REST because it was necessary for me to minimize possible URLs and to group certain URLs for auth in web server. So it is not a proper way to build a REST API service.

I consider Sinatra as a good solution for simple REST services and simple apps without unneeded Rails complexity. There a lot of API, auth, model plugins and Rack middleware to Sinatra so you could build your app from the ground. Sinatra uses Rack middleware to work with requests/responses so with some Rack magic we could distinguish stream against form (see two POST approaches) and store a file in Mongo and filesystem.

Omnifiles separates logic into different apps - public (for GET requests) and protected (POST and control panel). Protected app is using digest auth using `Rack::Auth`. Routing between two apps is performed by `Rack::URLMap`. It is definitely less flexible than Rails routes and unfortunately cannot route by HTTP methods.

For presentation I am using HAML. HAML is an another markup language above HTML and it is a lot more concise than ERB. It's required to learn it for a while it because it seems strange and awkward at first. A common problem with HAML is strict indent and string policy. Resulting markup is compact and beautiful and I like it.

App can be served as a thin app (there is a launcher script `bin/omnifiles`) or as a Rack app (using `config.ru`). Command line usage of thin or launcher can be cumbersome as you can see in [README.md](https://github.com/theirix/omnifiles/blob/master/README.md). Sometimes it is simpler to run Rack app inside Passenger.