FROM chancsu/yampdocker:orig

#AKA: alesssia/yampdocker:latest
#sha256:2081e7fe71843de6b7aef4f91f79abf27cce8157b6779ec64ef6d98561c1beca

#add in bin scripts from repo
RUN apk --update add --no-cache git
RUN git clone https://github.com/chan-csu/YAMP.git
ENV PATH="/YAMP/bin:${PATH}"

