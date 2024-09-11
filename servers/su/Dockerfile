# Stage 1: Build the dynamic binary
FROM rust:1.75.0 as builder

# Set the working directory
WORKDIR /usr/src/su

# Install required dependencies for building
RUN apt-get update && apt-get install -y \
    llvm-dev \
    libclang-dev \
    clang \
    librocksdb-dev

# Copy the manifests and cache dependencies
COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs && cargo build --release && rm -f target/release/deps/su*

# Copy the actual source code and build the application
COPY src ./src
COPY migrations ./migrations
RUN cargo build --release

# Stage 2: Create the runnable image using dynamic linking
FROM debian:bookworm-slim as runner

# Set the working directory
WORKDIR /app

# Copy the binary from the builder stage
COPY --from=builder /usr/src/su/target/release/su /app/su

# Install the necessary runtime libraries
RUN apt-get update && apt-get install -y libstdc++6 libssl3 libpq5 ca-certificates && rm -rf /var/lib/apt/lists/*

# Ensure the binary is executable
RUN chmod +x /app/su

# Run the binary
CMD ["/app/su"]

