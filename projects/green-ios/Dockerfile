# OSS Scanner image for Blockstream App iOS (https://github.com/Blockstream/green_ios).
# Blockstream App for iOS is an Xcode / UIKit / SwiftUI project targeting Apple platforms, so it
# requires macOS and Xcode SDKs to compile and link. This Linux image provides the Swift toolchain
# and repository source for static/source analysis and runs Linux-compatible checks.
FROM swift:6.0-bookworm

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates \
      git \
      jq \
      python3 \
      ripgrep \
    && rm -rf /var/lib/apt/lists/*

# scanner contract: the checkout lives inside the image, at /src
COPY . /src
WORKDIR /src

# Run repository checks that work on Linux without Xcode (non-fatal).
RUN python3 tools/check_localization.py || echo "WARNING: check_localization.py reported issues"
RUN find gaios core hw lightning NotificationService -name '*.swift' -print0 \
    | xargs -0 -n 50 swiftc -parse || echo "WARNING: swiftc -parse reported syntax warnings"
