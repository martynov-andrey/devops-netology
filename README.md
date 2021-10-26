# devops-netology
First edit

# Local .terraform directories
## Следующее правило игнорирует все файлы находящиеся в любых поддиректориях директории .terraform
**/.terraform/*

# .tfstate files
## Игнорировать все файлы с расширением .tfstate и все файлы содержащие в себе .tfstate перед любым расширением. Файлы игнорируются на любом уровне вложенности
*.tfstate
*.tfstate.*

# Crash log files
## Игнорировать конкретный файл crash.log где бы он не находился
crash.log

# Exclude all .tfvars files, which are likely to contain sentitive data, such as
# password, private keys, and other secrets. These should not be part of version
# control as they are data points which are potentially sensitive and subject
# to change depending on the environment.
#
## Рекурсивно игорировать все файлы с расширением .tfvars 
*.tfvars

# Ignore override files as they are usually used to override resources locally and so
# are not checked in
## Игнорировать следующие файлы override.tf, override.tf.json и все файлы с окончание именеми _override.tf, _override.tf.json. Правило для этих файлов действует рекурсивно
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Include override files you do wish to add to version control using negated pattern
#
# !example_override.tf

# Include tfplan files to ignore the plan output of command: terraform plan -out=tfplan
# example: *tfplan*

# Ignore CLI configuration files
## Игнорировать файлы .terraformrc и terraform.rc в любых директориях и поддиректориях
.terraformrc
terraform.rc
