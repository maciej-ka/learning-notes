Perl
====
test first:

```bash
perl -lane 'print "docker volume rm $F[1]"'
docker volume ls | grep local | docker volume ls | grep local | perl -lane 'print "docker volume rm $F[1]"'
```

run:

```bash
perl -lane 'system("docker volume rm $F[1]")'
docker volume ls | grep local | docker volume ls | grep local | perl -lane 'system("docker volume rm $F[1]")'
```



awk
===
run some command for each line of input

test it first:  
$2 means content of second word  
(awk splits each line into words by whitespace)

```bash
awk '{print "docker volume rm " $2}'
docker volume ls | grep local | docker volume ls | grep local | awk '{print "docker volume rm " $2}'
```

and then run:

```bash
awk '{system("docker volume rm " $2)}'
docker volume ls | grep local | docker volume ls | grep local | awk '{system("docker volume rm " $2)}'
```

