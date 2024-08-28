FROM public.ecr.aws/lambda/provided:al2023
RUN dnf update \
    && dnf install -y python3.11-3.11.6-1.amzn2023.0.3  \
    && dnf clean all \
    && /bin/python3.11 -m ensurepip --upgrade \
    && /bin/python3.11 -m pip install boto3
WORKDIR /code
COPY ./bootstrap ./bootstrap
COPY ./secret_manager.py ./secret_manager.py
ENTRYPOINT ["/code/bootstrap"]


