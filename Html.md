# Homeページ用

```HTML
<html>
    <head>
        <base target="_blank" />

        <style>
            div {
                float: left;
            }
            ul {
                width: 300px;
                margin: 0;
                padding: 0;
                list-style-type: none;
                background-color: #eee;
            }
            li a {
                font-size: 12px;
                display: block;
                padding: 4px 12px;
                text-decoration: none;
                color: #000;
            }
            li {
                text-align: left;
            }
            li:last-child {
                border-bottom: none;
            }
            li a.active {
                color: #fff;
                background-color: #da3c41;
            }
            li a:hover:not(.active) {
                color: #fff;
                background-color: #1b2538;
            }
        </style>
    </head>
    <body>
        <div>
            <ul>
                <li><a class="active" href="#">AAAA</a></li>
                <li><a href="">BBBB</a></li>
                <li><a href="">BBBB</a></li>
                <li><a href="">BBBB</a></li>
                <li><a href="">BBBB</a></li>
            </ul>
            <ul>
                <li><a class="active" href="#">AAAA</a></li>
                <li><a href="">BBBB</a></li>
                <li><a href="">BBBB</a></li>
                <li><a href="">BBBB</a></li>
                <li><a href="">BBBB</a></li>
            </ul>
        </div>
    </body>
</html>
```