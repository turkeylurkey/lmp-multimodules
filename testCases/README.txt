Test different project layouts
a)  pom, ear+war+jar
    -Server info in ear module
    -Main pom: ./pom.xml
a.2) pom folder at the same level as (sibling folder of) ear+war+jar
    -Server info in ear module
    -Main pom: ./pom/pom.xml
b)  pom, war+jar
    -Server info in war module
    -Main pom: ./pom.xml
c)  ear(MM)+war+jar
    -invalid: <module> requires packaging type pom
d)  war(MM)+jar
    -invalid: <module> requires packaging type pom
e)  pom, liberty-assembly+ear+war+jar
    -Server info in ear module
    -Main pom: ./pom.xml
f)  liberty-assembly+ear+war+jar
    -invalid: <module> requires packaging type pom
g)  pom,  server(pom)+ear+war+jar
    -Server info in pom module
    -Main pom: ./pom.xml
h)  server(pom)+ear+war+jar
    -invalid: <module> requires packaging type pom
i)   same as (a) but "ear" project has a <parent> which is not the "pom" project
    -Server info in ear module
    -Main pom: ./pom.xml

Test process
    - ensure hot deployment works
       - sources
       - resources
       - server config
    - see what happens with pressing Enter and/or hot tests
    - ensuring debug works on the different modules (works)
       - ensure hot code replacement works on the different module (works)
