---
layout: post
title: A faster way to replace an element's contents in a web page
description: "How I made a built in browser function 3000 times faster!"
headline: "Fast setting of innerHTML"
categories: javascript
tags: 
  - javascript
  - wtf
comments: false
mathjax: false
featured: true
published: true
---

The other day I was writing a web user interface in Javascript to view a dataset and I discovered that some calls to set innerHTML on a div element were taking over a minute (in Chrome). This line was the culprit:
{% highlight js %}
someDiv.innerHTML = newHTML;
{% endhighlight %}

Looks innocuous right? I dug a little deeper and discovered that the slow calls were only when the existing html in the div was large (~6M characters). The new html could be trivially small, and still the call was taking 1 minute. The existing html had a large number of elements with their onclick function set to some function on the document. It seemed like Chrome was taking a long time removing the exisiting elements from the div. I tried out an alternative to directly setting innerHTML.
{% highlight js %}
function replaceInnerHTML(oldDiv, html) {
    var newDiv = oldDiv.cloneNode(false);
    newDiv.innerHTML = html;
    oldDiv.parentNode.replaceChild(newDiv, oldDiv);
};
{% endhighlight %}
The original node is cloned to preserve all its properties except for it child nodes. Then we set the new node's innerHTML and replace references to the old node in its parent with the new node. This was just as fast as the direct method for setting a large html value, but up to 3000 times faster subsequently setting the html to something else! You can try it out below. The time taken to replace the contents of a div directly using innerHTML increases with the number of child elements currently in the div. The ratio shows just how much faster my method is. In Chrome I've seen it vary from 4 to 3000X faster, and up to 600X faster in Firefox. 
<br>
<br>Current large html size (characters): <span id="size">0</span>

<a class="btn btn-success btn-large" onclick="timeAndCompare();">Time setting innerHTML directly and my replacement method</a>
<a class="btn btn-danger btn-large" onclick="largeHtml += largeHtml;updateSize();">Double large html size</a>

<h3>Times taken (S)</h3>

<table>
    <col width="200">
    <col width="100">
    <col width="200">
    <col width="100">
    <tr>
        <td>Direct (large):</td>
        <td><span id="directL">0</span>

        </td>
        <td>Direct (small):</td>
        <td><span id="directS">0</span>

        </td>
    </tr>
    <tr>
        <td>Custom (large):</td>
        <td><span id="customL">0</span>

        </td>
        <td>Custom (small):</td>
        <td><span id="customS">0</span>

        </td>
    </tr>
</table>
<br>
<span id="speedup" style="color:red"></span>
<br>
<span style="width:900px; height:200px; overflow-y:auto; word-wrap:break-word;">
    <span id="box"></span>
</span>


<script>
var smallHtml = "Some text for the small html.";
var largeHtml = "";
for (var i = 0; i < 10000; i++) {
    largeHtml += "<span style='cursor:pointer; color:blue;'>name" + i + "</span>&nbsp;";
}
updateSize();

function updateSize() {
    document.getElementById("size").innerHTML = largeHtml.length;
};

function someFunction(stuff) {};

function timeAndCompare() {
    var t1 = time(setter(direct, largeHtml))();
    var t2 = time(setter(direct, smallHtml))();
    var t3 = time(setter(fast, largeHtml))();
    var t4 = time(setter(fast, smallHtml))();
    document.getElementById("directL").innerHTML = (t1) / 1000;
    document.getElementById("directS").innerHTML = (t2) / 1000;
    document.getElementById("customL").innerHTML = (t3) / 1000;
    document.getElementById("customS").innerHTML = (t4) / 1000;
    var ratio = Math.floor(t2/t4*10)/10;
    document.getElementById("speedup").innerHTML = "Speed up: " + ratio + ((ratio > 100) ? "!!!! Holy cow!" : ((ratio > 5) ? "!" : ""));
};

function time(f) {
    return function () {
        var t1 = Date.now();
        f();
        var t2 = Date.now();
        return t2 - t1;
    };
};

var setter = function (f, html) {
    return function () {
        f(document.getElementById("box"), html);
    };
};

var direct = function (element, html) {
    element.innerHTML = html;
};

var fast = function (oldElement, html) {
    var newElement = oldElement.cloneNode(false);
    newElement.innerHTML = html;
    oldElement.parentNode.replaceChild(newElement, oldElement);
};
</script>