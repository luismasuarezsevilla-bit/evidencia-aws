# Evidencia: ciclo de vida de un usuario IAM

- **Cuenta:** 559258952581
- **Usuario administrador:** `admin-luis-1`
- **Usuario de práctica:** `dev-luis` → renombrado a `lector-luis`
- **Política usada:** `AmazonEC2ReadOnlyAccess`

---

## Paso 3: Mostrar los datos del usuario y anotar su Arn

**Comando:**
```powershell
aws iam get-user --user-name dev-luis
```

**Arn del usuario (visible al crearlo, paso 1):** `arn:aws:iam::559258952581:user/dev-luis`

![Creación del usuario dev-luis](evidencia/paso01_crear_usuario.png)

> ⚠️ PENDIENTE: falta la captura de `get-user`. Solo existe `dev-luis` antes del renombrado, así que esta salida ya no se puede generar con ese nombre (ver nota al final).

---

## Paso 6: Verificar nombre nuevo y etiqueta

**Comandos:**
```powershell
aws iam update-user --user-name dev-luis --new-user-name lector-luis
aws iam tag-user --user-name lector-luis --tags Key=area,Value=desarrollo
aws iam get-user --user-name lector-luis
```

**Evidencia parcial** (nombre nuevo y Arn nuevo: `arn:aws:iam::559258952581:user/lector-luis`):

![Renombrado a lector-luis](evidencia/paso05_renombrado_lista_usuarios.png)

> ⚠️ PENDIENTE: falta la captura de `aws iam get-user --user-name lector-luis`, que debe mostrar el bloque `Tags` con `area = desarrollo`.

---

## Paso 8: Políticas adjuntas al usuario

**Comando:**
```powershell
aws iam list-attached-user-policies --user-name lector-luis
```

**Salida:**
```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonEC2ReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess"
        }
    ]
}
```

![Política adjunta](evidencia/paso08_politica_adjunta.png)

---

## Paso 12: Acción permitida (`ec2 describe-instances`)

**Comando:**
```powershell
aws ec2 describe-instances --profile lector
```

**Salida** (responde sin error; la cuenta no tiene instancias):
```json
{
    "Reservations": []
}
```

![Acción permitida y denegada](evidencia/paso12_13_permitida_y_denegada.png)

---

## Paso 13: Acción NO permitida (`iam list-users`)

**Comando:**
```powershell
aws iam list-users --profile lector
```

**Salida:**
```
aws: [ERROR]: An error occurred (AccessDenied) when calling the ListUsers operation:
User: arn:aws:iam::559258952581:user/lector-luis is not authorized to perform:
iam:ListUsers on resource: arn:aws:iam::559258952581:user/ because no
identity-based policy allows the iam:ListUsers action.
```

(Misma captura del paso 12, arriba.)

---

## Paso 15: Repetir el paso 12 después de quitar la política

**Comandos:**
```powershell
aws iam detach-user-policy --user-name lector-luis --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
aws ec2 describe-instances --profile lector
```

**Salida:**
```
aws: [ERROR]: An error occurred (UnauthorizedOperation) when calling the
DescribeInstances operation: You are not authorized to perform this operation.
User: arn:aws:iam::559258952581:user/lector-luis is not authorized to perform:
ec2:DescribeInstances because no identity-based policy allows the
ec2:DescribeInstances action
```

![Acción denegada tras quitar la política](evidencia/paso15_tras_quitar_politica.png)

---

## Paso 19: Listar usuarios y confirmar que ya no aparece

**Comando:**
```powershell
aws iam list-users
```

> ⚠️ PENDIENTE: el usuario `lector-luis` todavía no se ha eliminado (ver pasos 16 a 18).

---

## Reto: borrar el usuario antes de quitar sus recursos

Al intentar `aws iam delete-user --user-name lector-luis` con recursos asociados, IAM respondió:

```
An error occurred (DeleteConflict) when calling the DeleteUser operation:
Cannot delete entity, must detach all policies first.
```

IAM no permite borrar un usuario que todavía tiene políticas, llaves de acceso u otros elementos asociados.
