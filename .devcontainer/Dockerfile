# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

# Install asdf-vm
ARG ASDF_VERSION=0.18.0
ARG ASDF_ARCH=linux-arm64
ARG ASDF_TARBALL=asdf-v${ASDF_VERSION}-${ASDF_ARCH}.tar.gz
ARG ASDF_URL=https://github.com/asdf-vm/asdf/releases/download/v${ASDF_VERSION}/${ASDF_TARBALL}
ARG ASDF_MD5_URL=${ASDF_URL}.md5
RUN curl -fsSLo /tmp/${ASDF_TARBALL} ${ASDF_URL} \
  && curl -fsSLo /tmp/${ASDF_TARBALL}.md5 ${ASDF_MD5_URL} \
  && ASDF_MD5="$(awk '{print $NF}' /tmp/${ASDF_TARBALL}.md5)" \
  && echo "${ASDF_MD5}  /tmp/${ASDF_TARBALL}" | md5sum -c - \
  && mkdir -p /opt/asdf \
  && tar xvz -C /opt/asdf -f /tmp/${ASDF_TARBALL} \
  && rm -f /tmp/${ASDF_TARBALL} /tmp/${ASDF_TARBALL}.md5
ENV PATH="/opt/asdf/asdf/bin:${PATH}"

ARG USERNAME=vscode
USER $USERNAME

# select zsh as default shell for $USERNAME
RUN sudo chsh -s $(which zsh) $USERNAME
SHELL ["/bin/zsh", "-i", "-c"]

# select agnoster as oh-my-zsh theme
RUN omz theme set agnoster

# Configure custom prompt
RUN echo "prompt_context() { prompt_segment black default '👴' }" >> ~/.zshrc

# install zsh-syntax-highlighting plugin
RUN git clone --branch 0.8.0 --depth 1 https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting \
  && omz plugin enable zsh-syntax-highlighting

# install zsh autosuggestions plugin
RUN git clone --branch v0.7.1 --depth 1 https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions \
  && omz plugin enable zsh-autosuggestions

# Enable oh-my-zsh asdf plugin
RUN omz plugin enable asdf
