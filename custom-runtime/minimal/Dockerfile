FROM public.ecr.aws/lambda/provided:al2023
RUN dnf update && dnf install -y python3.11-3.11.6-1.amzn2023.0.3  && dnf clean all
WORKDIR /code
COPY ./bootstrap ./bootstrap
ENTRYPOINT ["/code/bootstrap"]


