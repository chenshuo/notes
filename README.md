# Shuo's notes
http://chenshuo.github.io/notes

```bash
$ python3 -m pip install mkdocs python-markdown-math mkdocs-graphviz mkdocs-jupyter jupyter_contrib_nbextensions notebook==6.5.5

Also modify mkdocs_jupyter/nbconvert2.py

@@ -226,13 +233,13 @@

     if no_input:
         template_exported_conf.update(
             {
                 "TemplateExporter": {
                     "exclude_output_prompt": True,
-                    "exclude_input": True,
+                    # "exclude_input": True,
                     "exclude_input_prompt": True,
                 }
             }
         )


```
