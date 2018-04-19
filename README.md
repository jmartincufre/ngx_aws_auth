# AWS proxy module + hacks

[![Build Status](https://travis-ci.org/anomalizer/ngx_aws_auth.svg?branch=master)](https://travis-ci.org/anomalizer/ngx_aws_auth)
 [![Gitter chat](https://badges.gitter.im/anomalizer/ngx_aws_auth.png)](https://gitter.im/ngx_aws_auth/Lobby?utm_source=share-link&utm_medium=link&utm_campaign=share-link)

This nginx module can proxy requests to authenticated S3 backends using Amazon's
V4 authentication API. The first version of this module was written for the V2
authentication protocol and can be found in the *AuthV2* branch.

This fork includes support for static access to s3 in addition to signed access. This fork just tracks master
of https://github.com/anomalizer/ngx_aws_auth. thanks to https://github.com/acejam for this patch

Note: this version is less secure than dynamic signing. If you are using it you should rotating your aws
credentials regularly.

## License
This project uses the same license as ngnix does i.e. the 2 clause BSD / simplified BSD / FreeBSD license

## Usage example

A basic nginx config using static auth credentials looks like:


```nginx
env S3_BUCKET;
env AWS_ACCESS_KEY;
env AWS_SECRET_KEY;

http {

    proxy_cache_lock on;
    proxy_cache_lock_timeout 5s;
    proxy_cache_path /etc/nginx/cache levels=1:2 keys_zone=s3cache:10m max_size=30g;

    server {
      listen 80;
      server_name localhost;

      location / {
        set_by_lua $s3_bucket 'return os.getenv("S3_BUCKET")';
        set_by_lua $aws_access_key 'return os.getenv("AWS_ACCESS_KEY")';
        set_by_lua $aws_secret_key 'return os.getenv("AWS_SECRET_KEY")';

        s3_bucket $s3_bucket;
        aws_access_key $aws_access_key;
        aws_secret_key $aws_secret_key;

        resolver 8.8.8.8;
        proxy_pass https://$s3_bucket.s3.amazonaws.com;
        proxy_set_header Authorization $s3_auth_token;
        proxy_set_header x-amz-date $aws_date;

        proxy_cache s3cache;
        proxy_cache_valid 200 302 24h;
        proxy_cache_valid 404 1m;
        proxy_cache_convert_head on;
        add_header X-Proxy-Cache $upstream_cache_status;
      }
    }

```

See: https://github.com/anomalizer/ngx_aws_auth for more info

## Known limitations
The 2.x version of the module currently only has support for GET and HEAD calls. This is because
signing request body is complex and has not yet been implemented.



## Credits
Original idea based on http://nginx.org/pipermail/nginx/2010-February/018583.html and suggestion of moving to variables rather than patching the proxy module.

Subsequent contributions can be found in the commit logs of the project.
