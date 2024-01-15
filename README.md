## Generating the Datasheet
Compile it using TexLive

## Generating the Webpage
Installing mkdocs : ```pip install mkdocs```

Installing different extensions : 
- Math expression support: ```pip install python-markdown-math```
- Plotting figures :
    -  ```pip3 install mkdocs-charts-plugin``` (you might need to use version 0.0.6)
    - ```pip install pymdown-extensions```
    - ```pip install Pygments```

For pdf export (not working on Windows): 
- ```pip install mkdocs-with-pdf```
- ```pip install weasyprint==52.2```
- ```sudo apt install libpango1.0-dev```

mermaid support : 
```pip install mkdocs-mermaid2-plugin ```