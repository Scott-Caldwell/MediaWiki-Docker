FROM mediawiki:1.35

# Using this file as a base https://github.com/pastakhov/compose-mediawiki-ubuntu/blob/master/web/Dockerfile

# Environment variables
ENV MW_VERSION=REL1_35 \
    lists=/var/www/html \
    MW_HOME=/var/www/html \
    DEBIAN_FRONTEND=noninteractive

RUN mkdir /var/log/mediawiki
RUN chown www-data:www-data /var/log/mediawiki/

# Turn off SSL Verify to work with firewall rules
RUN git config --global http.sslVerify false

RUN \
    apt-get update && \
    apt-get install libldap2-dev wget libpng-dev sendemail sendmail less vim gnupg2 libzip-dev zip unzip ghostscript imagemagick poppler-utils -y && \
    rm -rf /var/lib/apt/lists/*

RUN \
    echo 'deb https://packages.tideways.com/apt-packages-main any-version main' | tee /etc/apt/sources.list.d/tideways.list && \
    wget -qO - https://packages.tideways.com/key.gpg | apt-key add - && \
    apt-get update && \
    apt-get install tideways-daemon tideways-php tideways-cli -y && \
    rm -rf /var/lib/apt/lists/*

# Install PHP modules
RUN docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/
RUN docker-php-ext-install ldap
RUN docker-php-ext-install gd
RUN docker-php-ext-configure zip
RUN docker-php-ext-install zip

# Mpdf extension for PDFs
RUN set -x; \
    cd $MW_HOME/extensions \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-Mpdf.git Mpdf \
    && cd Mpdf \
	&& wget https://getcomposer.org/composer.phar \
	&& php composer.phar install --no-dev

# Other extensions
RUN set -x; \
    cd $MW_HOME/extensions \
    && git clone https://github.com/Scott-Caldwell/mediawiki-extensions-Favorites.git Favorites && (cd Favorites && git reset --hard b30b2429140b19a9b33435eb61370180e8730611) \
    && git clone --depth 1 https://github.com/lightbinder/Html5mediator.git Html5mediator \
    && git clone --depth 1 https://github.com/sorvis/HideCategorizedOrphans.git \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-CSS.git CSS \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-MsUpload.git MsUpload \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-ArticleToCategory2.git ArticleToCategory2 \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-Math.git Math \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-RSS.git RSS \
    && git clone https://github.com/sigbertklinke/Iframe.git && (cd Iframe && git reset --hard a8b4d1f3c2f7743061ca168c5a2aca553aa648fd) \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-MultiBoilerplate.git MultiBoilerplate \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-Survey.git Survey \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-Quiz.git Quiz \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-CodeMirror.git CodeMirror \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-MassEditRegex.git MassEditRegex \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-JsonConfig.git JsonConfig \
    && git clone --depth 1 -b sorvis_REL1_35 https://github.com/sorvis/mediawiki-extensions-Graph.git Graph \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-HitCounters.git HitCounters \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-TitleKey.git TitleKey \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-PluggableAuth.git PluggableAuth \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-LDAPProvider.git LDAPProvider \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-LDAPAuthentication2.git LDAPAuthentication2 \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-EventLogging.git EventLogging \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-EventStreamConfig.git EventStreamConfig \
    && git clone --depth 1 -b 0.9.2 https://github.com/samwilson/diagrams-extension.git Diagrams \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-LabeledSectionTransclusion.git LabeledSectionTransclusion \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-ProofreadPage.git ProofreadPage\
    && git clone https://github.com/Scott-Caldwell/ViewStats.git ViewStats && (cd ViewStats && git reset --hard 163a2ca7e564d4ee66396a7dec2c45b536b37233) \
    && git clone --depth 1 -b Fix_broken_extension https://github.com/sorvis/mediawiki-extensions-SearchStats.git SearchStats \
    && git clone --depth 1 -b master https://github.com/sorvis/mw-markdown.git Markdown && (cd Markdown && git reset --hard a48077fca11b420a39d8dead0e7213da00654a1b && git clone -b master https://github.com/erusev/parsedown.git && cd parsedown && git checkout 1.7.4 && mv Parsedown.php ../.) \
    && git clone --depth 1 -b $MW_VERSION https://github.com/wikimedia/mediawiki-extensions-WikidataPageBanner.git WikidataPageBanner

COPY ./images/fav-icons.png $MW_HOME/extensions/Favorites/resources/images/fav-icons.png

# Run maintenance scripts
# Increase value for run maintenance script before web service started
ENV MW_AUTOUPDATE=true \
    MW_MAINTENANCE_UPDATE=3 \
    MW_MAINTENANCE_CIRRUSSEARCH_UPDATECONFIG=1 \
    MW_MAINTENANCE_CIRRUSSEARCH_FORCEINDEX=1 \
    MW_MAINTENANCE_ULS_INDEXER=1 \
    MW_SCRIPT_CLDR_REBUILD=1 \
    MW_SITE_NAME=ScriptMed\ Wiki \
    MW_SITE_LANG=en \
    MW_DEFAULT_SKIN=vector \
    MW_MAIN_CACHE_TYPE=CACHE_ACCEL \
    MW_REST_DOMAIN=localhost \
    MW_REST_RESTBASE_PROXY_PATH=/api/rest_ \
    MW_REST_RESTBASE_PORT=7231 \
    MW_SHOW_EXCEPTION_DETAILS=true \
    PHP_LOG_ERRORS=On \
    PHP_ERROR_REPORTING=E_ALL

RUN set -x; mkdir /tmp/tideways && cd /tmp/tideways && git clone https://github.com/tideways/php-xhprof-extension.git \
    && cd php-xhprof-extension \
    && phpize \
    && ./configure \
    && make \
    && make install
