Name
    nginx_limit_access_module - support to deny request by specific variable
    with HTTP POST interface

Status
    This module is at its very early phase of development and considered
    highly experimental. But you're encouraged to test it out on your side
    and report any quirks that you experience.

    We need your help! If you find this module useful and/or interesting,
    please consider joining the development!

Synopsis
  a simple deny ip example:
    http {

        limit_access_zone  zone=one:5m bucket_number=10007 type=ip;

        server {
            listen       80;
            server_name  localhost;

            limit_access_variable zone=one $limit_access_deny;

            location / {
                root   html;
                index  index.html index.htm;

                if ($limit_access_deny) {
                    return 403;
                }
            }

            location /limit_interface {
                allow   192.168.1.0/24;
                deny all;
                limit_access_interface zone=one;
            }
        }
    }

  deny both $remote_addr and $host example:
    http {

        limit_access_zone  zone=addr:5m bucket_number=10007 type=$remote_addr;
        limit_access_zone  zone=host:5m bucket_number=10007 type=$host;

        server {
            listen       80;
            server_name  _;

            limit_access_variable zone=addr $limit_access_deny_addr;
            limit_access_variable zone=host $limit_access_deny_host;

            location / {
                root   html;
                index  index.html index.htm;

                if ($limit_access_deny_addr) {
                    return 403;
                }

                if ($limit_access_deny_host) {
                    return 403;
                }
            }

            location /limit_interface_addr {
                allow   192.168.1.0/24;
                deny all;
                limit_access_interface zone=addr;
            }

            location /limit_interface_host {
                allow   192.168.1.0/24;
                deny all;
                limit_access_interface zone=host;
            }
        }
    }

Description
    This module can deny request specific variable with HTTP POST interface.

Directives
  limit_access_zone
    syntax: *limit_access_zone zone=name:size [bucket_number=number]
    [type=ip|$your_variable];*

    default: *none*

    context: *http*

    The directive describes the area, which stores the record of the limit
    hash table. Example of usage:

    limit_access_zone zone=one:5m bucket_number=10007 type=ip;

    In this case, the hash table is allocated 5MB as a zone called "one".
    The hash table's bucket number is 10007. We can get the record's key of
    this hash table from the remainder $binary_remote_addr with
    bucket_number.

    If the record is the type of ip, each record's size is about 32 bytes.
    If you have a 5M zone, it can hold about 100k+ records.

    The type can be 'ip' or any of the variable in Nginx. I use the 'ip' as
    a special variable while I treat the ip as an integer. This can save
    some storage memory. Otherwise, I treat all the variable to be string.

  limit_access_interface
    syntax: *limit_access_interface zone=name;*

    default: *none*

    context: *location*

    This directive set the interface location where you can add or delete
    the deny list. See the section of Interface for detail.

  limit_access
    syntax: *limit_access zone=name;*

    default: *none*

    context: *http, server, location*

    This directive set the location where you can deny specific variable
    from the zone's record. If you want to do deny action in several
    locations, you can use this directive in their parent block. The
    directive is like the deny
    (<http://wiki.nginx.org/HttpAccessModule#deny>).

    It will look up the zone's hash table. If it find the request's variable
    is in the hash table and it's not expired, then return 403.

  limit_access_variable
    syntax: *limit_access_variable zone=name $limit_access_variable_name;*

    default: *none*

    context: *http, server, location*

    This directive set the name of variable and the zone attached with the
    variable. When you access this variable, the variable will do like the
    limit_access directive. It will look up the zone's hash table. If it
    finds the request's variable is in the hash table and it's not expired,
    then the value is '1'. If not, the value is null.

    If you want to do the same deny action in several locations, you can use
    the directive of *limit_acccess*.

  limit_access_default_expire
    syntax: *limit_access_default_expire expire_time;*

    default: *1d*

    context: *http, server, location*

    Set the default expire time for the ban list record. Default expire time
    is 1 day. The unit of time can be: s(second,default), m(minute),
    h(hour), d(day), w(week), M(month), y(year).

  limit_access_log_level
    syntax: *limit_access_log_level info|notice|warn|error;*

    default: *limit_access_log_level error*

    context: *http, server, location*

    Control the log level of the request deny message.

Interface
    The request method sent to the interface location is POST, and the
    content-type is application/x-www-form-urlencoded. The content is like
    this:

    ban_type=variable&ban_expire=3600&ban_list=192.168.1.1,192.168.1.2

    The parameters' specification is:

    ban_type: the type of ban_list. It can be: ip, variable. It's related
    with the type of zone.

    ban_expire: the expire time for the ban list. The unit of time can be:
    s(second,default), m(minute), h(hour), d(day), w(week), M(month),
    y(year).

    ban_list: the ban list, the ipv4 address can also be sent by a binary
    form.

    free_type: the type of free_list.

    free_list: free the ban list, the variable will not be denied any more.

    show_type: the type of show_list.

    show_list: show the ban list. You can get the specific ban list like
    this:

    show_type=variable&show_list=127.0.0.1,127.1.1.1

    If your zone's type is IP, and you want to show all the current ban IP
    list, You can do like this:

    show_type=ip&show_list=all

    destroy_list: invalidate all the current ban list, then all the client
    request is allowed.

Installation
    Download the latest version of the release tarball of this module from
    github (<http://github.com/yaoweibin/nginx_limit_access_module>)

    Grab the nginx source code from nginx.org (<http://nginx.org/>), for
    example, the version 0.8.53 (see nginx compatibility), and then build
    the source with this module:

        $ wget 'http://nginx.org/download/nginx-0.8.53.tar.gz'
        $ tar -xzvf nginx-0.8.53.tar.gz
        $ cd nginx-0.8.53/

        $ ./configure --add-module=/path/to/nginx_limit_access_module

        $ make
        $ make install

Compatibility
    My test bed 0.8.53.

TODO
Known Issues
    Developing

Changelogs
  v0.1
    first release

Authors
    Weibin Yao(姚伟斌) *yaoweibin AT gmail DOT com*

License
    This README template is from agentzh (<http://github.com/agentzh>).

    This module is licensed under the BSD license.

    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions are
    met:

    Redistributions of source code must retain the above copyright notice,
    this list of conditions and the following disclaimer.
    Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
    IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED
    TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
    PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
    HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
    SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED
    TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
    PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
    LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
    NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
    SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

