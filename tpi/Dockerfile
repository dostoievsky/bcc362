FROM alpine:latest

ARG NAME
ARG CLASS

RUN apk add --no-cache openssl && \
    mkdir $NAME && \
    echo "Name: $NAME\nClass: $CLASS" > $NAME/$NAME.txt && \
    openssl genrsa -out $NAME/chave_$NAME.key 2048 && \
    openssl rsa -in $NAME/chave_$NAME.key -pubout -out $NAME/chave_$NAME.pub && \
    openssl dgst -sha256 -hex $NAME/$NAME.txt > $NAME/hash256_$NAME.txt && \
    openssl rsautl -encrypt -pubin -inkey $NAME/chave_$NAME.pub -in $NAME/$NAME.txt | openssl base64 > $NAME/$NAME.txt.enct && \
    openssl rsautl -decrypt -inkey $NAME/chave_$NAME.key -in $NAME/$NAME.txt.enct | openssl base64 -d > $NAME/$NAME.recuperado.txt

WORKDIR $NAME

CMD ["/bin/sh"]

