# 5.6 Paquetes

1. El máximo de paquetes almacenados por usuario es 10.
2. Abrir un paquete reduce el total en 1.
3. La regeneración ocurre cada 1 minuto.
4. Nunca se supera el máximo de 10.
5. Si el usuario ya posee 10, no se genera deuda de regeneración.
6. La cantidad debe poder calcularse de forma consistente incluso si el usuario estuvo desconectado.
7. El usuario obtiene 5 cartas por paquete abierto.
8. Mientras el usuario no haya verificado su email, su cantidad de paquetes es siempre 0 y no hay regeneración.
9. Al verificar el email (por enlace o al iniciar sesión con Google), el usuario recibe 10 paquetes y la regeneración comienza desde ese momento.
