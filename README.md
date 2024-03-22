import socket
import threading
import time

# Función para enviar un mensaje broadcast de solicitud de conexión
def buscar_clientes():
    while True:
        sock.sendto(b'ping', ('255.255.255.255', 9999))
        time.sleep(2)

# Función para manejar las respuestas de los clientes
def manejar_respuestas():
    while True:
        try:
            data, addr = sock.recvfrom(1024)
            if data == b'ping':
                usuarios_conectados.add(addr[0])  # Añadir solo la dirección IP al conjunto
            else:
                print(f"Recibido de {addr[0]}: {data.decode()}")  # Imprimir el mensaje recibido
        except ConnectionResetError:
            # Manejar desconexión de un cliente
            if addr[0] in usuarios_conectados:
                print(f"Cliente {addr[0]} se ha desconectado.")
                usuarios_conectados.remove(addr[0])
                # Notificar a los demás clientes sobre la desconexión
                for usuario in usuarios_conectados:
                    sock.sendto(f"El cliente {addr[0]} se ha desconectado.".encode(), (usuario, 9999))

# Configuración del socket UDP
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(('0.0.0.0', 9999))
sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)

# Conjunto para almacenar direcciones de usuarios conectados
usuarios_conectados = set()

# Crear hilos para buscar clientes y manejar respuestas
thread_buscar = threading.Thread(target=buscar_clientes)
thread_buscar.daemon = True
thread_buscar.start()

thread_respuestas = threading.Thread(target=manejar_respuestas)
thread_respuestas.daemon = True
thread_respuestas.start()

# Bucle principal para la interfaz de usuario
while True:
    try:
        opcion = input("Ingrese 'listar' para mostrar usuarios conectados, 'enviar' para enviar un mensaje, o 'desconectar' para cerrar el servidor: ")
        
        if opcion == 'listar':
            print("Lista de usuarios conectados:")
            for usuario in usuarios_conectados:
                print(usuario)
             # Eliminar clientes que no responden (por inactividad) después de cierto tiempo
            clientes_a_eliminar = set()
            for cliente in usuarios_conectados:
                if cliente not in usuarios_conectados:
                    clientes_a_eliminar.add(cliente)
            for cliente in clientes_a_eliminar:
                usuarios_conectados.remove(cliente)
                print(f"Cliente {cliente} desconectado por inactividad.")
                
                # Esperar antes de volver a comprobar

            time.sleep(2)  

            # Función para actualizar la lista de usuarios conectados
          def actualizar_lista_usuarios():
    global usuarios_conectados
    # Inicializa la lista de usuarios conectados como un conjunto vacío
    usuarios_conectados = set()

# Aquí comienza el bloque principal del programa
while True:
    try:
        opcion = input("Ingrese 'listar' para ver la lista de usuarios conectados, 'enviar' para enviar un mensaje o 'desconectar' para cerrar el servidor: ")
        
        if opcion == 'listar':
            # Aquí se listarán los usuarios conectados
            print("Usuarios conectados:")
            for usuario in usuarios_conectados:
                print(usuario)
        
        elif opcion == 'enviar':
            # Se solicita la dirección IP del destinatario y el mensaje a enviar
            destinatario = input("Ingrese la dirección IP del destinatario: ")
            mensaje = input("Ingrese el mensaje a enviar: ")
            if destinatario in usuarios_conectados:
                # Si el destinatario está en la lista de usuarios conectados, se envía el mensaje
                # utilizando sockets UDP en el puerto 9999
                sock.sendto(mensaje.encode(), (destinatario, 9999))
                print(f"Mensaje enviado a {destinatario}: {mensaje}")
            else:
                # Si el destinatario no está en la lista de usuarios conectados, se imprime un mensaje de error
                print("El destinatario no está conectado.")
        
        elif opcion == 'desconectar':
            # Aquí se notifica a todos los clientes que el servidor se está desconectando
            for usuario in usuarios_conectados:
                sock.sendto(b"El servidor se ha desconectado.", (usuario, 9999))
            # Se cierra el socket del servidor y se sale del bucle
            sock.close()
            break
    
    except EOFError:
        # Maneja la excepción EOFError, que se produce cuando se alcanza el final de un archivo de entrada
        print("Entrada no aceptada. Utilice 'listar', 'enviar' o 'desconectar'.")

