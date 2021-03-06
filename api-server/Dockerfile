# Build a virtualenv using the appropriate Debian release:
# * Install python3-venv for the built-in Python3 venv module (not installed by default).
# * Install gcc libpython3-dev to compile C Python modules.
# * Update pip to support bdist_wheel.
FROM debian:buster-slim AS dependencies
RUN apt-get update && \
    apt-get install --no-install-suggests --no-install-recommends --yes gcc python3-venv libpython3-dev && \
    python3 -m venv /venv && \
    /venv/bin/python -m pip install --upgrade pip

# Build the virtualenv as a separate step to only re-execute this step when "requirements.txt" changes.
FROM dependencies AS build
COPY requirements.txt /requirements.txt
RUN /venv/bin/python -m pip install -r /requirements.txt

# Copy the virtualenv into a distroless image.
# For more information, please refer to https://github.com/GoogleContainerTools/distroless.
FROM gcr.io/distroless/python3-debian10 AS run
COPY --from=build /venv /venv
ADD hypercorn.toml /app/hypercorn.toml
ADD app.py /app/app.py
ADD routers/ /app/routers/
WORKDIR /app
EXPOSE 5000
ENTRYPOINT [ "/venv/bin/python", "-m", "hypercorn", "-c", "hypercorn.toml", "app:app" ]
