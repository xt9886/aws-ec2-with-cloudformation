# Crear el Role `MyAdminEc2Role` con permisos de superadmin

Entrar a AWS IAm y crar ese rol.

# Crear el `ec2-windows-template.yml`

```yml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: ami-04f77c9cd94746b09
      KeyName: my-key-pair
      SecurityGroupIds:
        - !GetAtt
          - MySecurityGroup
          - GroupId
      Tags:
        - Key: Name
          Value: MyInstance
  MyIamInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName: MyIamInstanceProfile
      Path: "/"
      Roles:
        - MyAdminEc2Role
  MyLaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: MyLaunchTemplate
      LaunchTemplateData:
        IamInstanceProfile:
          Arn: !GetAtt
            - MyIamInstanceProfile
            - Arn
        DisableApiTermination: true
        ImageId: ami-04f77c9cd94746b09
        InstanceType: t3.micro
        KeyName: my-key-pair
        SecurityGroupIds:
          - !GetAtt
            - MySecurityGroup
            - GroupId
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access via port 80
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3389
          ToPort: 3389
          CidrIp: 0.0.0.0/0

# Output section to display the Launch Template ID and the Public IP address of the instance
Outputs:
  InstanceId:
    Description: The Instance ID
    Value: !Ref MyInstance
  PublicIp:
    Description: The Public IP address of the instance
    Value: !GetAtt MyInstance.PublicIp
```

# Crear el par de claves

```bash

aws ec2 create-key-pair --key-name my-key-pair --query 'KeyMaterial' --output text > my-key-pair.pem

```

# Desplegar la plantilla EC2

```bash
aws cloudformation create-stack --stack-name my-windows-ec2-stack --template-body file://ec2-windows-template.yml --capabilities CAPABILITY_NAMED_IAM
```

# Verificar el estado de la pila

```bash
aws cloudformation describe-stacks --stack-name my-windows-ec2-stack --query "Stacks[0].StackStatus"
```

# Obtener la IP Pública de la instancia

````bash
aws cloudformation describe-stacks --stack-name my-windows-ec2-stack --query "Stacks[0].Outputs[?OutputKey=='PublicIp'].OutputValue" --output text
```
````

````bash
aws cloudformation describe-stacks --stack-name my-windows-ec2-stack --query "Stacks[0].Outputs[?OutputKey=='InstanceId'].OutputValue" --output text
```
````

# Obtener la contraseña de administrador

```bash
aws ec2 get-password-data --instance-id <INSTANCE_ID> --priv-launch-key my-key-pair.pem --query "PasswordData" --output text
```

```markdown
> **NOTA:** Reemplaza `<INSTANCE_ID>` con el ID de la instancia que obtuviste en el Paso 5.
```

# Conectarse a la instancia

1. Abre la aplicación Conexión a Escritorio Remoto en tu computadora personal.

2. Ingresa la IP pública de la instancia.

3. Cuando te solicite credenciales, usa:

   - Usuario: Administrator

   - Contraseña: La contraseña que obtuviste en el Paso 6.

# Al terminar la práctica, elimina la cola

```bash
aws cloudformation delete-stack --stack-name my-windows-ec2-stack
```
