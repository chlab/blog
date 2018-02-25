---
layout: post
title: Manually parse raw HTTP data with PHP
categories:
- Web development
tags:
- php
---
How to parse multipart data manually in PHP when working with HTTP PUT requests.

Luckily, PHP offers some great tools for working with data that we usually take for granted. When posting multipart form-data to a PHP script, the POST values will automatically become available as an array of values in the *$_POST* superglobal, the files will automatically be processed and be made available in the *$_FILES* superglobal. Unfortunately, this doesn’t work for PUT requests. This is probably by design, as with a PUT request you are actually sending a specific file to a specific location on a server. This is supported, as is sending an url-encoded querystring as the payload. What doesn’t work out of the box though is sending multipart data in a PUT request.

Consider the following pseudo code on the client side:

{% highlight php %}
curl_setopt_array(
  CURLOPT_POSTFIELDS => array(
    'user_id' => 3,
    'post_id' => 5,
    'image' => '@/tmp/current_file'),
  CURLOPT_CUSTOMREQUEST => 'PUT'
  );
{% endhighlight %}

If you get rid of the *CURLOPT_CUSTOMREQUEST* bit, this will be sent as a POST, and the data will be available in the *$_POST* and *$_FILES* arrays respectively. With PUT, the data will not be parsed by PHP at all. So I wrote a method to do it manually. Unparsed multipart data looks something like this:

{% highlight text %}
------------------------------b2449e94a11c
Content-Disposition: form-data; name="user_id"

3
------------------------------b2449e94a11c
Content-Disposition: form-data; name="post_id"

5
------------------------------b2449e94a11c
Content-Disposition: form-data; name="image"; filename="/tmp/current_file"
Content-Type: application/octet-stream

�����JFIF���������... a bunch of binary data
{% endhighlight %}

Here’s a method that will parse this:

{% highlight php %}
/**
 * Parse raw HTTP request data
 *
 * Pass in $a_data as an array. This is done by reference to avoid copying
 * the data around too much.
 *
 * Any files found in the request will be added by their field name to the
 * $data['files'] array.
 *
 * @param   array  Empty array to fill with data
 * @return  array  Associative array of request data
 */
function parse_raw_http_request(array &$a_data)
{
  // read incoming data
  $input = file_get_contents('php://input');

  // grab multipart boundary from content type header
  preg_match('/boundary=(.*)$/', $_SERVER['CONTENT_TYPE'], $matches);

  // content type is probably regular form-encoded
  if (!count($matches))
  {
    // we expect regular puts to containt a query string containing data
    parse_str(urldecode($input), $a_data);
    return $a_data;
  }

  $boundary = $matches[1];

  // split content by boundary and get rid of last -- element
  $a_blocks = preg_split("/-+$boundary/", $input);
  array_pop($a_blocks);

  // loop data blocks
  foreach ($a_blocks as $id => $block)
  {
    if (empty($block))
      continue;

    // you'll have to var_dump $block to understand this and maybe replace \n or \r with a visibile char

    // parse uploaded files
    if (strpos($block, 'application/octet-stream') !== FALSE)
    {
      // match "name", then everything after "stream" (optional) except for prepending newlines
      preg_match("/name=\"([^\"]*)\".*stream[\n|\r]+([^\n\r].*)?$/s", $block, $matches);
      $a_data['files'][$matches[1]] = $matches[2];
    }
    // parse all other fields
    else
    {
      // match "name" and optional value in between newline sequences
      preg_match('/name=\"([^\"]*)\"[\n|\r]+([^\n\r].*)?\r$/s', $block, $matches);
      $a_data[$matches[1]] = $matches[2];
    }
  }
}
{% endhighlight %}

And here’s how to use it:

{% highlight php %}
$a_data = array();
parse_raw_http_request($a_data);
var_dump($a_data);
{% endhighlight %}

In the above example, the *var_dump* would look something like this:

{% highlight php %}
Array (
  [user_id] => 3
  [post_id] => 5
  [files] => Array (
    [image] => [binary data]
  )
)
{% endhighlight %}

I hope this helps.

**Update**

I'm happy to see that a few people have used the code above and some have changed it to meet their requirements. Check out [commenter Jas’ version](https://gist.github.com/jas-/5c3fdc26fedd11cb9fb5#file-stream-php) for multiple file and input-type support.
