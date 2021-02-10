FROM chancsu/yampdocker:orig
#AKA: alesssia/yampdocker:latest
#sha256:2081e7fe71843de6b7aef4f91f79abf27cce8157b6779ec64ef6d98561c1beca

#add in bin scripts from repo
RUN apk --update add --no-cache git
RUN git clone https://github.com/chan-csu/YAMP.git
ENV PATH="/YAMP/bin:${PATH}"

#conda install -c bioconda -y bbmap=37.10 fastqc=0.11.5 metaphlan2=2.6.0 qiime=1.9.1 humann2=0.9.9 export2graphlan:0.20-py27_0 graphlan:1.1.3-1 hclust2:0.99-py_2 

#RUN echo "backend: Agg" > ~/.config/matplotlib/matplotlibrc
