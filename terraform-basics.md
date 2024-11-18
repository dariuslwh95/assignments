----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# variable manipulation
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## lists
- have an index starting from 0
```
variable prefix {
	default = ["Mr", "Mrs", Sir"]
	type = list
}
```

## map
- you can set type restrictions on the maps
- note counts of a list are more favourable than map
```
variable file-content {
	type = map
	default = {
		"statement 1" = "value1"
		"statement 2" = "value2"
	}
}
```

## objects
- custom data types
```
variable "bella" {
	type = object({
		name = string
		color = string
		age = number
		food = list(string)
		favourite_pet = bool
	})
	default = {
		...
	}
}
```

## tuples
- a list with mixed data types and validation
```
variable kitty {
	type = tuple([string, number, bool])
	default = [x , 1, true]
}
```

## TF_VAR + terraform.tfvars
- you can export variables to the terminal session
- or use terraform.tfvars, terraform.tfvars.json files to hold variables
- -var-file can also be used <*.tfvars>
```
TF_VAR_<variable name>
```

## variable precedence
1. Command-line flags (-var or -var-file)
2. Environment variables (TF_VAR_)
3. terraform.tfvars or terraform.tfvars.json files
4. *Any .auto.tfvars or *.auto.tfvars.json files (processed in lexical order)
5. Default values in variable definitions

## resource attributes
- referenced by using <resource type>.<resource name>.<resource attribute>
```
resource "time_static" "time_update" {
}
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# terraform state
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

- --refresh=false flag at terraform apply can be useful to improve performance
- each user should have the latest state file always
- remote data store, should not run terraform at the same time

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# terraform commands
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## validation
terraform graph


## lifecycle rules
resource "aws_instnce" "webserver" {
	ami = "ami-91923812093"
	instance_type = "t2.micro"
	tags = {
		Name = "ProjectA-Webserver"
	}
	lifecycle = {
		# create_before_destroy = 
		# prevent_destroy = 
		# ignore_changes =
	}
}

