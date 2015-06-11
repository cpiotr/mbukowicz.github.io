---
layout: post
title: PHP REST client with Pest
date: '2013-11-11T15:54:00.001+01:00'
tags: 
modified_time: '2013-11-12T09:19:50.321+01:00'
thumbnail: http://4.bp.blogspot.com/-CWF0OqyWRZI/Unvu-XwGKII/AAAAAAAAAJ4/KzKTfSlfh9w/s72-c/php-med-trans.png
blogger_id: tag:blogger.com,1999:blog-7932117927902690732.post-9061903315236332880
blogger_orig_url: http://www.code-thrill.com/2013/11/php-rest-client-with-pest.html
---

<h2>Introduction</h2> <div class="separator" style="clear: both; text-align: center;"><a href="http://4.bp.blogspot.com/-CWF0OqyWRZI/Unvu-XwGKII/AAAAAAAAAJ4/KzKTfSlfh9w/s1600/php-med-trans.png" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><img border="0" src="http://4.bp.blogspot.com/-CWF0OqyWRZI/Unvu-XwGKII/AAAAAAAAAJ4/KzKTfSlfh9w/s1600/php-med-trans.png" /></a></div> <p>In this post I'm going to show a very nice and simple way of creating PHP clients to RESTful web services. For this we are going to use a library called <a href="https://github.com/educoder/pest">Pest</a>, available on GitHub.</p> <a name='more'></a> <h2>Prerequisities</h2> <p><a href="https://github.com/educoder/pest">Pest</a> requires the <strong>mod_curl</strong> extension, so please make sure it is properly installed on your web server before proceeding further.</p> <h2>Project structure</h2> <p>The next step is to download the library files and put them in a separate folder called <strong>/Pest</strong>. We will also create an example <strong>index.php</strong> script, where we will put several code snippets calling a REST service. Here is the final project structure:</p> <pre class="brush:plain; gutter:false;"><br />|<br />+- Pest/<br />|  |<br />|  +- Pest.php<br />|  +- PestJSON.php<br />|  +- PestXML.php<br />|<br />+- index.php<br /></pre> <h2>Include <strong>Pest</strong></h2> <p>At the beginning of the <strong>index.php</strong> lets include the <strong>Pest</strong> library:</p> <pre class="brush:php; gutter:false;"><br />&lt;?php<br /><br />require_once 'Pest/Pest.php';<br />require_once 'Pest/PestJSON.php';<br />require_once 'Pest/PestXML.php';<br /></pre> <h2>Plain HTTP</h2> <p>We will start with a simple example of calling our service using plain HTTP <strong>GET</strong> method:</p> <pre class="brush:php; gutter:false;"><br />$address = "http://127.0.0.1:8080";<br />$pest = new Pest($address);<br />$result = $pest->get("/my/rest/uri");<br /></pre> <p>In this example the <strong>$result</strong> variable contains a raw string, so data processing must be manually done by the programmer. It means that you will have to implement unmarshalling logic, for example from JSON, XML, YAML, etc., all by yourself.</p> <p>Now, lets see how we can <strong>POST</strong> some data and headers:</p> <pre class="brush:php; gutter:false;"><br />$data = array(<br /> "i" => 123,<br /> "str" => "Bob",<br /> "arr" => array("a", "b", "c"),<br /> "map" => array(<br />  "a" => 1,<br />  "b" => 2,<br />  "c" => 3<br /> )<br />);<br /><br />$headers = array(<br /> "abc" => "123"<br />);<br /><br />$result = $pest->post("/my/rest/uri", $data, $headers);<br /></pre> <p>In the service data is available through the parameters map:</p> <pre class="brush:plain; gutter:false"><br />i -> 123<br />str -> Bob<br />arr[0] -> a<br />arr[1] -> b<br />arr[2] -> c<br />map[a] -> 1<br />map[b] -> 2<br />map[c] -> 3<br /></pre> <p>...and headers are available through the headers map:</p> <pre class="brush:plain; gutter:false"><br />abc -> 123<br /></pre> <p>Again, we have to manually process the <strong>$result</strong>. This is a tedious task and a programmer shouldn't be involved in the most cases. In the following examples we will look into ways of automatically marshalling/unmarshalling data to/from JSON/XML.</p> <h2>JSON</h2> <p>Supposedly, we have a REST service accessible by calling <strong>GET</strong> method on the uri <strong>/my/rest/uri.json</strong> returning this data:</p> <pre class="brush:plain; gutter:false"><br />{ <br />  "i": 42, <br />  "str": "Alice", <br />  "arr": ["d", "e", "f"], <br />  "map": { "x": 123, "y": 456 } <br />}<br /></pre> <p>We can "turn on" the automatic unmarshalling from JSON to PHP objects tree by replacing the <strong>Pest</strong> object with <strong>PestJSON</strong>:</p> <pre class="brush:php; gutter:false;"><br />$address = "http://127.0.0.1:8080";<br />$pest = new PestJSON($address);<br />$result = $pest->get("/my/rest/uri.json");<br />print_r($result);<br /></pre> <p>The upper script prints out:</p> <pre class="brush:plain; gutter:false"><br />Array ( <br />  [i] => 42 <br />  [str] => Alice <br />  [arr] => Array ( [0] => d [1] => e [2] => f ) <br />  [map] => Array ( [x] => 123 [y] => 456 ) <br />)<br /></pre> <p>With switching to the <strong>PestJSON</strong> we also get automatic marshalling to JSON. If we <strong>POST</strong> this data to our REST service:</p> <pre class="brush:php; gutter:false;"><br />$data = array(<br /> "i" => 123,<br /> "str" => "Bob",<br /> "arr" => array("a", "b", "c"),<br /> "map" => array(<br />  "a" => 1,<br />  "b" => 2,<br />  "c" => 3<br /> )<br />);<br />$result = $pest->post("/my/rest/uri.json", $data);<br /></pre> <p>...on the server in the body we receive:</p><pre class="brush:plain; gutter:false"><br />{"i":123,"str":"Bob","arr":["a","b","c"],"map":{"a":1,"b":2,"c":3}}<br /></pre> <h2>XML</h2> <p>A similar case is with the XML format. Lets create a REST service accessible by calling <strong>GET</strong> on <strong>/my/rest/uri.xml</strong>. The service returns this XML:</p> <pre class="brush:xml; gutter:false"><br />&lt;grandparent&gt;<br />  &lt;parent&gt;<br />    &lt;child&gt;<br />      &lt;i&gt;123&lt;/i&gt;<br />      &lt;str&gt;Frank&lt;/str&gt;<br />      &lt;list&gt;<br />        &lt;item&gt;abc&lt;/item&gt;<br />        &lt;item&gt;def&lt;/item&gt;<br />        &lt;item&gt;ghi&lt;/item&gt;<br />      &lt;/list&gt;<br />      &lt;map&gt;<br />        &lt;item key="a"&gt;1&lt;/item&gt;<br />        &lt;item key="b"&gt;2&lt;/item&gt;<br />        &lt;item key="c"&gt;3&lt;/item&gt;<br />      &lt;/map&gt;<br />    &lt;/child&gt;<br />  &lt;/parent&gt;<br />&lt;/grandparent&gt;<br /></pre> <p>Again, lets call it from Pest by replacing the <strong>Pest</strong> class with <strong>PestXML</strong> subclass:</p> <pre class="brush:php; gutter:false;"><br />$address = "http://127.0.0.1:8080";<br />$pest = new PestXML($address);<br />$result = $pest->get("/my/rest/uri.xml");<br />print_r($result);<br /></pre> <p>The script echoes the following PHP object tree:</p> <pre class="brush:plain; gutter:false"><br />SimpleXMLElement Object ( <br />  [parent] => SimpleXMLElement Object ( <br />    [child] => SimpleXMLElement Object ( <br />      [i] => 123 <br />      [str] => Frank <br />      [list] => SimpleXMLElement Object ( <br />        [item] => Array ( <br />          [0] => abc <br />          [1] => def <br />          [2] => ghi <br />        ) <br />      ) <br />      [map] => SimpleXMLElement Object ( <br />        [item] => Array ( <br />          [0] => 1 <br />          [1] => 2 <br />          [2] => 3 <br />        ) <br />      ) <br />    ) <br />  ) <br />)<br /></pre> <p>To access response we can either traverse the object tree in the standard PHP way:</p> <pre class="brush:php; gutter:false;"><br />$i = (int) $result->parent->child->i;<br /></pre> <p>...or by using XPath:</p> <pre class="brush:php; gutter:false;"><br />$queryResult = $result->xpath("/grandparent/parent/child/i");<br />$i = (int) $queryResult[0];<br /></pre> <p>The first example looks simpler, however XPath allows us to execute more advanced queries. What's more, we don't have to test the nullity of objects - the query will just return empty results array. In the first example, if parent or child was absent, an exception is thrown.</p> <p>Lets take a closer look at a case where XPath seems more suitable. The following code extracts the item with key equal to "a":</p> <pre class="brush:php; gutter:false;"><br />$queryResult = $result->xpath("/grandparent/parent/child/map/item[@key='a']");<br />$item = $queryResult[0];<br />$key = (string) $item['key'];<br />$value = (int) $item;<br />echo "$key -> $value";<br /></pre> <p>The result:</p> <pre class="brush:plain; gutter:false"><br />a -> 1<br /></pre> <p>Lets finish with a little gotcha (which may be fixed in the future). If we <strong>POST</strong> this data:</p> <pre class="brush:php; gutter:false;"><br />$data = array(<br /> "i" => 123,<br /> "str" => "Bob",<br /> "arr" => array("a", "b", "c"),<br /> "map" => array(<br />  "a" => 1,<br />  "b" => 2,<br />  "c" => 3<br /> )<br />);<br /><br />$result = $pest->post("/my/rest/uri.xml", $data);<br /></pre> <p>...on the server side the request will have empty body and data is passed like in plain HTTP (through parameters map):</p> <pre class="brush:plain; gutter:false"><br />i -> 123<br />str -> Bob<br />arr[0] -> a<br />arr[1] -> b<br />arr[2] -> c<br />map[a] -> 1<br />map[b] -> 2<br />map[c] -> 3<br /></pre> <h2>The End</h2> <p>I hope you enjoyed this post and as me you found the <a href="https://github.com/educoder/pest">Pest</a> library to be simple and robust.</p>