### Reflected DOM XSS
With this code, we can set a home page:
```html
<!DOCTYPE html>
<html>
    <body>
        <h1> Eval </h1>
        <p id="page"></p>
    </body>
<script>

var xmlhttp= new XMLHttpRequest();
var url = 'http://localhost:8000/data.json';

xmlhttp.onreadystatechange = function(){
    if (this.readyState == 4 && this.status == 200){
        console.log(this.response);

        // let myObj = JSON.parse(this.response);   this is the correct line of code!
        eval('let myObj = ' + this.response);   // This is the insecure one. Because bypasses the JSON.parse.
        
        
        console.log(typeof myObj);
        document.getElementById('page').innerText = myObj.information;
    }
}

xmlhttp.open("GET", url, true);
xmlhttp.send();


</script>
</html>
```

And this is our data.json file:
```json
{"information":"batman"-alert)(1)}
```

We are bypassing the JSON.parse, and input our data directly into an eval. This is where the vulnerability arises. 
