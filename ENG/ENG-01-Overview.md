##### Other languages: [简体中文](/CHN/CHN-01-概述)

# Overview

**Drogon** is a C++17/20-based HTTP application framework. Drogon can be used to easily build various types of web application server programs using C++.

**Drogon** is the name of a dragon in the American TV series "Game of Thrones" that I really like.

Drogon's main application platform is Linux. It also supports Mac OS, FreeBSD and Windows.

Its main features are as follows:

* Use a non-blocking I/O network lib based on epoll (kqueue under macOS/FreeBSD) to provide high-concurrency, high-performance network IO, please visit the [TFB Tests Results](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite) for more details;
* Provide a completely asynchronous programming mode;
* Support Http1.0/1.1 (server side and client side);
* Based on template, a simple reflection mechanism is implemented to completely decouple the main program framework, controllers and views.
* Support cookies and built-in sessions;
* Support back-end rendering, the controller generates the data to the view to generate the Html page. Views are described by CSP template files, C++ codes are embedded into Html pages through CSP tags. And the drogon command-line tool automatically generates the C++ code files for compilation;
* Support view page dynamic loading (dynamic compilation and loading at runtime);
* Provide a convenient and flexible routing solution from the path to the controller handler;
* Support filter chains to facilitate the execution of unified logic (such as login verification, Http Method constraint verification, etc.) before handling HTTP requests;
* Support https (based on OpenSSL);
* Support WebSocket (server side and client side);
* Support JSON format request and response, very friendly to the Restful API application development;
* Support file download and upload;
* Support gzip, brotli compression transmission;
* Support pipelining;
* Provide a lightweight command line tool, drogon_ctl, to simplify the creation of various classes in Drogon and the generation of view code;
* Support non-blocking I/O based asynchronously reading and writing database (PostgreSQL and MySQL(MariaDB) database);
* Support asynchronously reading and writing sqlite3 database based on thread pool;
* Support ARM Architecture;
* Provide a convenient lightweight ORM implementation that supports for regular object-to-database bidirectional mapping;
* Support plugins which can be installed by the configuration file at load time;
* Support AOP with build-in joinpoints.

# Next: [Install drogon](/ENG/ENG-02-Installation)
