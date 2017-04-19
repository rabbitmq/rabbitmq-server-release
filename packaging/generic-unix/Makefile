SOURCE_DIST_FILE ?= $(wildcard ../../../rabbitmq-server-*.tar.xz)

ifneq ($(filter-out clean,$(MAKECMDGOALS)),)
ifeq ($(SOURCE_DIST_FILE),)
$(error Cannot find source archive; please specify SOURCE_DIST_FILE)
endif
ifneq ($(words $(SOURCE_DIST_FILE)),1)
$(error Multiple source archives found; please specify SOURCE_DIST_FILE)
endif

VERSION ?= $(patsubst rabbitmq-server-%.tar.xz,%,$(notdir $(SOURCE_DIST_FILE)))
ifeq ($(VERSION),)
$(error Cannot determine version; please specify VERSION)
endif
endif

ORIG_SOURCE_DIR = rabbitmq-server-$(VERSION)
SOURCE_DIR = pivotal-rabbitmq-server-$(VERSION)
TARGET_DIR = pivotal-rabbitmq_server-$(VERSION)
TARGET_TARBALL = pivotal-rabbitmq-server-generic-unix-$(VERSION)

unexport DEPS_DIR
unexport ERL_LIBS
MAKEOVERRIDES =

all: dist
	@:

dist:
	xzcat $(SOURCE_DIST_FILE) | tar -xf -
	mv $(ORIG_SOURCE_DIR) $(SOURCE_DIR)
	cd $(SOURCE_DIR); patch -p1 < ../../commercial-patches/rabbitmq-server-$(VERSION)-pivotal.patch
# web-manpages are not used by generic-unix but by `make release` in the.
# Umbrella. Those manpages are copied to www.rabbitmq.com
	$(MAKE) -C $(SOURCE_DIR) \
		PREFIX= RMQ_ROOTDIR= \
		RMQ_ERLAPP_DIR=`pwd`/$(TARGET_DIR) \
		MANDIR=`pwd`/$(TARGET_DIR)/share/man \
		manpages web-manpages install install-man

	sed -e 's:^SYS_PREFIX=$$:SYS_PREFIX=\$${RABBITMQ_HOME}:' \
        $(TARGET_DIR)/sbin/rabbitmq-defaults >$(TARGET_DIR)/sbin/rabbitmq-defaults.tmp \
        && mv $(TARGET_DIR)/sbin/rabbitmq-defaults.tmp $(TARGET_DIR)/sbin/rabbitmq-defaults
	chmod 0755 $(TARGET_DIR)/sbin/rabbitmq-defaults

	cp ../OSL/RabbitMQ_OSL.txt $(TARGET_DIR)/RabbitMQ_$(VERSION)_OSL.txt
	cp ../EULA/EULA_Pivotal_RabbitMQ.txt $(TARGET_DIR)/EULA_Pivotal_RabbitMQ_$(VERSION).txt
	cp README.pivotal $(TARGET_DIR)

	mkdir -p $(TARGET_DIR)/etc/rabbitmq

	find $(TARGET_DIR) -print0 | LC_COLLATE=C sort -z | \
	xargs -0 tar --no-recursion -cf - | \
	xz > $(CURDIR)/$(TARGET_TARBALL).tar.xz

	if test "$(PACKAGES_DIR)"; then \
		mkdir -p "$(PACKAGES_DIR)"; \
		mv $(TARGET_TARBALL).tar.xz "$(PACKAGES_DIR)"; \
	fi

clean: clean_partial
	rm -f pivotal-rabbitmq-server-generic-unix-*.tar.xz

clean_partial:
	rm -rf pivotal-rabbitmq-server-* pivotal-rabbitmq_server-*
