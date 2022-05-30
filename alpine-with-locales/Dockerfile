FROM alpine:3.16.0

ENV MUSL_LOCPATH="/usr/share/i18n/locales/musl"

RUN apk --no-cache add libintl && \
    apk --no-cache --virtual .locale_build add cmake make musl-dev gcc gettext-dev git && \
    git clone https://gitlab.com/rilian-la-te/musl-locales && \
    cd musl-locales && \
    cmake -DLOCALE_PROFILE=OFF -DCMAKE_INSTALL_PREFIX:PATH=/usr . && \
    make && make install && \
    cd .. && \
    rm -r musl-locales && \
    apk del .locale_build make make musl-dev gcc gettext-dev git



#BUILD
#docker build  --rm -f "alpine-with-locales\Dockerfile" -t alpine-localized:3.16 "alpine-with-locales"