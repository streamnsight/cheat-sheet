# OCI Terrafrom provider tips and tricks

## Extract compartment resources

```bash
terraform-provider-oci -command=export -compartment_name=<compartment_name> -output_path=./
```

