ARG BASE_IMAGE=jupyter/r-notebook:1386e2046833
FROM $BASE_IMAGE

USER root

ENV PATH=$PATH:/usr/lib/rstudio-server/bin \
    R_HOME=/opt/conda/lib/R
ARG LITTLER=$R_HOME/library/littler

RUN \
    # download R studio
    curl --silent -L --fail https://s3.amazonaws.com/rstudio-ide-build/server/bionic/amd64/rstudio-server-1.2.1578-amd64.deb > /tmp/rstudio.deb && \
    echo '81f72d5f986a776eee0f11e69a536fb7 /tmp/rstudio.deb' | md5sum -c - && \
    \
    # install R studio
    apt-get update && \
    apt-get install -y --no-install-recommends /tmp/rstudio.deb && \
    rm /tmp/rstudio.deb && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    \
    # setting default CRAN mirror
    echo "local({\n" \
         "   r <- getOption('repos')\n" \
         "   r['CRAN'] <- 'https://cloud.r-project.org'\n" \
         "   options(repos = r)\n" \
         "})\n" > $R_HOME/etc/Rprofile.site && \
    \
    # littler provides install2.r script
    R -e "install.packages(c('littler', 'docopt'))" && \
    \
    # modifying littler scripts to conda R location
    sed -i 's/\/usr\/local\/lib\/R\/site-library/\/opt\/conda\/lib\/R\/library/g' \
        ${LITTLER}/examples/*.r && \
	ln -s ${LITTLER}/bin/r ${LITTLER}/examples/*.r /usr/local/bin/ && \
	echo "$R_HOME/lib" | sudo tee -a /etc/ld.so.conf.d/littler.conf && \
	ldconfig
    
USER $NB_USER

RUN pip install nbgitpuller okpy && \
    pip install git+https://github.com/okpy/jassign.git && \
    pip install jupyter-server-proxy jupyter-rsession-proxy && \
    \
    # remove cache
    rm -rf ~/.cache/pip ~/.cache/matplotlib ~/.cache/yarn && \
    conda clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

