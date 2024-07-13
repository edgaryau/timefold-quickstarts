# syntax=docker/dockerfile:1

# Comments are provided throughout this file to help you get started.
# If you need more help, visit the Dockerfile reference guide at
# https://docs.docker.com/go/dockerfile-reference/

ARG PYTHON_VERSION=3.12
FROM python:${PYTHON_VERSION}-slim AS base

COPY --from=amazoncorretto:22-jdk /usr/lib/jvm/java-22-amazon-corretto /usr/lib/jvm/java-22-amazon-corretto
ENV JAVA_HOME=/usr/lib/jvm/java-22-amazon-corretto
RUN update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-22-amazon-corretto/bin/java 1

# Prevents Python from writing pyc files.
ENV PYTHONDONTWRITEBYTECODE=1

# Keeps Python from buffering stdout and stderr to avoid situations where
# the application crashes without emitting any logs due to buffering.
ENV PYTHONUNBUFFERED=1

WORKDIR /app
RUN chmod -R 755 /app

RUN mkdir /home/appuser

# Create a non-privileged user that the app will run under.
# See https://docs.docker.com/go/dockerfile-user-best-practices/
ARG USER_UID=10001
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/home/appuser" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${USER_UID}" \
    appuser

RUN chown -R appuser /home/appuser

# Download dependencies as a separate step to take advantage of Docker's caching.
# Leverage a cache mount to /root/.cache/pip to speed up subsequent builds.
# Leverage a bind mount to requirements.txt to avoid having to copy them into
# into this layer.
RUN --mount=type=cache,target=/root/.cache/pip \
    --mount=type=bind,source=requirements.txt,target=requirements.txt \
    python -m pip install -r requirements.txt

RUN python -m venv .venv
RUN . .venv/bin/activate

# USER appuser

# Copy the source code into the container.
COPY . .

RUN pip install -e .

# Switch to the non-privileged user to run the application.
USER appuser

# Expose the port that the application listens on.
EXPOSE 8080

# Run the application.
CMD ["run-app"]
