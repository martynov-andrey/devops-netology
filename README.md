### devops-netology
First edit

Second edit

Next edit

End of edit

*Следующее правило игнорирует все файлы находящиеся в любых поддиректориях директории .terraform*


        **/.terraform/*

*Игнорировать все файлы с расширением .tfstate и все файлы содержащие в себе .tfstate перед любым расширением. Файлы игнорируются на любом уровне вложенности*

        *.tfstate
        *.tfstate.*

*Игнорировать конкретный файл crash.log где бы он не находился*

        crash.log

*Рекурсивно игорировать все файлы с расширением .tfvars* 

        *.tfvars

*Игнорировать следующие файлы override.tf, override.tf.json и все файлы с окончание именеми _override.tf, _override.tf.json. Правило для этих файлов действует рекурсивно*

        override.tf
        override.tf.json
        *_override.tf
        *_override.tf.json

*Игнорировать файлы .terraformrc и terraform.rc в любых директориях и поддиректориях*

        .terraformrc
        terraform.rc
