## <a name="origin-secrets" id="origin-secrets" data-magellan-target="origin-secrets">Working with Origin Secrets</a>

With the Habitat CLI and a Builder account, you can encrypt and store secrets to expose at build time as environment variables in your Builder builds.  This feature is helpful for plans requiring access to protected resources at build time, such as private source-code repositories, cloud storage providers and the like. Secrets are defined at the origin level, which makes them usable in any plan belonging to that origin.

To work with origin secrets, first obtain a [Builder access token](#builder-token), then apply it on the command line using either the `HAB_AUTH_TOKEN` environment variable or the `--auth` option, along with the associated origin. For example, to list the names of all secrets defined for a given origin, use the `list` subcommand:

```
hab origin secret list --origin <ORIGIN> --auth <TOKEN>
```

You can also set your origin and token as environment variables:

```
export HAB_ORIGIN=<ORIGIN>
export HAB_AUTH_TOKEN=<TOKEN>
hab origin secret list
```

To create a new secret, give the secret a name and string value:

```
hab origin secret upload AWS_ACCESS_KEY_ID your-key-id
hab origin secret upload AWS_SECRET_ACCESS_KEY your-secret-access-key
```

Once your secret has been uploaded, you can refer to it in your plan file as an environment variable, and Builder will supply its decrypted value during your build job.

For instance, if a plan required access to a file kept in a private bucket on Amazon S3, you might use the AWS CLI provided by the `core/awscli` package to download the file using secrets named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, which would allow the `aws` binary to read them as environment variables at build time:

```
...
pkg_build_deps=(core/awscli)

do_download() {
  # When present, the AWS CLI will use the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
  # environment variables to authenticate with Amazon S3.
  aws s3 cp s3://your-private-bucket/your-file.tar.gz .
}
...
```

You can delete a secret either with the Habitat CLI:

```
hab origin secret delete AWS_ACCESS_KEY_ID
hab origin secret delete AWS_SECRET_ACCESS_KEY
```

or in the Builder web interface, under the Settings tab for your origin:

![Builder origin secrets](/images/screenshots/origin-secrets.png)

> Secrets are encrypted locally using an origin encryption key. Their values are readable only by Builder.
