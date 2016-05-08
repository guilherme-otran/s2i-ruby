FROM openshift/base-centos7

# This image provides a Ruby 2.3 environment you can use to run your Ruby
# applications.

MAINTAINER SoftwareCollections.org <sclorg@redhat.com>

EXPOSE 8080

ENV RUBY_VERSION 2.3

LABEL summary="Platform for building and running Ruby 2.3 applications" \
      io.k8s.description="Platform for building and running Ruby 2.3 applications" \
      io.k8s.display-name="Ruby 2.3" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,ruby,ruby23"

RUN yum install -y centos-release-scl

RUN yum install -y procps

RUN INSTALL_PKGS="autoconf \
		bison \
		bzip2 \
		bzip2-devel \
		ca-certificates \
		coreutils \
		curl \
		gcc \
		gdbm-devel \
		glib-devel \
		glibc-devel \
		libffi-devel \
		libxml2-devel \
		libxslt-devel \
		kernel-headers \
		make \
		ncurses-devel \
		openssl-devel \
		readline-devel \
		ruby \
		libyaml-devel \
		zlib-devel \
		php-devel \
		php-pear \
		ImageMagick \
		ImageMagick-devel \
		v8314 \
		nodejs010" && \
     yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
     rpm -V $INSTALL_PKGS && \
     yum clean all -y

# Compile ruby from source
ENV RUBY_MAJOR 2.3
ENV RUBY_VERSION 2.3.1
ENV RUBY_DOWNLOAD_SHA256 b87c738cb2032bf4920fef8e3864dc5cf8eae9d89d8d523ce0236945c5797dcd
ENV RUBYGEMS_VERSION 2.6.4

# # https://bugs.ruby-lang.org/issues/11869 and https://github.com/docker-library/ruby/issues/75
RUN curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
	&& echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
	&& mkdir -p /usr/src \
	&& tar -xzf ruby.tar.gz -C /usr/src \
	&& mv "/usr/src/ruby-$RUBY_VERSION" /usr/src/ruby \
	&& rm ruby.tar.gz \
	&& cd /usr/src/ruby \
	&& { echo '#define ENABLE_PATH_CHECK 0'; echo; cat file.c; } > file.c.new && mv file.c.new file.c \
	&& autoconf \
	# the configure script does not detect isnan/isinf as macros
	&& ac_cv_func_isnan=yes ac_cv_func_isinf=yes \
		./configure --disable-install-doc \
	&& make -j"$(getconf _NPROCESSORS_ONLN)" \
	&& make install \
	&& runDeps="$( \
		scanelf --needed --nobanner --recursive /usr/local \
			| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
			| sort -u \
			| xargs -r apk info --installed \
			| sort -u \
	)" \
	&& gem update --system $RUBYGEMS_VERSION \
	&& rm -r /usr/src/ruby

ENV BUNDLER_VERSION 1.12.2

RUN gem install bundler --version "$BUNDLER_VERSION"

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Each language image can have 'contrib' a directory with extra files needed to
# run and build the applications.
COPY ./contrib/ /opt/app-root

RUN chown -R 1001:0 /opt/app-root

USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
