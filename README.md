@startuml
skinparam classAttributeIconSize 0
skinparam nodesep 100
skinparam ranksep 120
skinparam backgroundColor #F9F9F9
skinparam class {
    BackgroundColor White
    ArrowColor #2C3E50
    BorderColor #2C3E50
}

' --- ENUMERACIONES COMPLETAS ---
enum EstadoHabitacion {
    DISPONIBLE
    OCUPADA
    MANTENIMIENTO
    RESERVADA
}

enum EstadoReserva {
    ACTIVA
    CANCELADA
    FINALIZADA
}

enum MetodoPersistencia {
    ARCHIVO_TXT
    ARCHIVO_BINARIO
    SERIALIZACION
}

' --- CAPA DE MODELO: CLIENTES (IMPLEMENTANDO INTERFAZ) ---
interface ICliente {
    + getId(): String
    + getNombre(): String
    + getTipoCliente(): String
    + getDatosContacto(): String
}

abstract class Cliente implements ICliente {
    # id: String
    # nombre: String
    # telefono: String
    # email: String
    # direccion: String
    + getId(): String
    + getNombre(): String
    + setNombre(n: String): void
    + getTelefono(): String
    + setTelefono(t: String): void
    + getEmail(): String
    + getDireccion(): String
    + {abstract} getTipoCliente(): String
    + getDatosContacto(): String
}

abstract class PersonaNatural extends Cliente {
    # cedula: String
    # apellidos: String
    # fechaNacimiento: String
    + getCedula(): String
    + getApellidos(): String
    + getNombreCompleto(): String
}

class PersonaNaturalHabitual extends PersonaNatural {
    - porcentajeDescuento: double
    - fechaRegistro: String
    - totalEstadias: int
    + getTipoCliente(): String
    + getDescuento(): double
    + setPorcentajeDescuento(p: double): void
    + incrementarEstadias(): void
}

class PersonaNaturalEsporadica extends PersonaNatural {
    + getTipoCliente(): String
}

abstract class Empresa extends Cliente {
    # nit: String
    # razonSocial: String
    # representanteLegal: String
    # sector: String
    + getNit(): String
    + getRazonSocial(): String
    + getRepresentanteLegal(): String
}

class EmpresaHabitual extends Empresa {
    - porcentajeDescuento: double
    - convenio: String
    - fechaConvenio: String
    + getTipoCliente(): String
    + getDescuento(): double
    + setPorcentajeDescuento(p: double): void
}

class EmpresaEsporadica extends Empresa {
    + getTipoCliente(): String
}

' --- CAPA DE MODELO: HABITACIONES ---
abstract class Habitacion {
    # numero: int
    # descripcion: String
    # precioPorNoche: double
    # disponible: boolean
    # estado: EstadoHabitacion
    + getNumero(): int
    + getDescripcion(): String
    + getPrecioPorNoche(): double
    + isDisponible(): boolean
    + setDisponible(d: boolean): void
    + setPrecioPorNoche(p: double): void
    + {abstract} getTipo(): String
    + {abstract} toString(): String
}

class HabitacionSencilla extends Habitacion {
    - tieneTv: boolean
    - tieneWifi: boolean
    + getTipo(): String
    + tieneTv(): boolean
    + tieneWifi(): boolean
    + toString(): String
}

class HabitacionDoble extends Habitacion {
    - numeroCamas: int
    - tieneVista: boolean
    + getTipo(): String
    + getNumeroCamas(): int
    + tieneVista(): boolean
    + toString(): String
}

class HabitacionMatrimonial extends Habitacion {
    - tieneJacuzzi: boolean
    - tieneTerrazaPrivada: boolean
    + getTipo(): String
    + tieneJacuzzi(): boolean
    + tieneTerrazaPrivada(): boolean
    + toString(): String
}

' --- CAPA DE MODELO: SERVICIOS Y FACTURACIÓN ---
class ServicioHotel {
    - idServicio: String
    - nombre: String
    - costo: double
    - categoria: String
    - descripcion: String
    - disponible: boolean
    + getIdServicio(): String
    + getNombre(): String
    + getCosto(): double
    + setCosto(c: double): void
    + isDisponible(): boolean
    + setDisponible(d: boolean): void
    + toString(): String
}

class ServicioConsumido {
    - fecha: String
    - cantidad: int
    - observaciones: String
    - servicio: ServicioHotel
    + getFecha(): String
    + getCantidad(): int
    + getSubtotal(): double
    + getServicio(): ServicioHotel
}

class FacturaInformativa {
    - idFactura: String
    - fechaEmision: String
    - observaciones: String
    - reserva: Reserva
    + getFactura(): String
    + getFechaEmision(): String
    + getCostoHabitaciones(): double
    + getDescuentoAplicado(): double
    + getCostoServicios(): double
    + getSubtotal(): double
    + getTotal(): double
    + getDetalleHabitaciones(): String
    + getDetalleServicios(): String
    + generarResumen(): String
    + guardarEnArchivo(ruta: String): void
}

' --- CAPA DE MODELO: RESERVA (CLAVE) ---
class Reserva {
    - idReserva: String
    - fechaInicio: String
    - numeroDias: int
    - estado: EstadoReserva
    - cliente: Cliente
    - habitaciones: List<Habitacion>
    - servicios: List<ServicioConsumido>
    + getIdReserva(): String
    + getFechaInicio(): String
    + getNumeroDias(): int
    + getEstado(): EstadoReserva
    + setEstado(e: EstadoReserva): void
    + getFechaFin(): String
    + agregarHabitacion(h: Habitacion): void
    + eliminarHabitacion(num: int): void
    + agregarServicioConsumido(sc: ServicioConsumido): void
    + calcularCostoHabitaciones(): double
    + calcularCostoServicios(): double
    + calcularTotal(): double
    + hayConflictoFechas(otra: Reserva): boolean
    + toString(): String
}

' --- CAPA DE PERSISTENCIA (DAO) SEGÚN image_9f3fbb.png ---
interface IDAO<T> {
    + guardar(entidad: T): void
    + eliminar(id: String): boolean
    + buscarPorId(id: String): T
    + listarTodo(): List<T>
}

abstract class BaseDAO {
    # rutaArchivo: String
    # metodoPersistencia: MetodoPersistencia
}

class ClienteDAO extends BaseDAO implements IDAO {
    + guardar(cliente: Cliente): void
    + eliminar(id: String): boolean
    + buscarPorId(id: String): Cliente
    + listarTodo(): List<Cliente>
}

class ReservaDAO extends BaseDAO implements IDAO {
    + guardar(reserva: Reserva): void
    + eliminar(id: String): boolean
    + buscarPorId(id: String): Reserva
    + listarTodo(): List<Reserva>
}

class HabitacionDAO extends BaseDAO implements IDAO {
    + guardar(habitacion: Habitacion): void
    + eliminar(id: String): boolean
    + buscarPorId(id: String): Habitacion
    + listarTodo(): List<Habitacion>
}

class ServicioDAO extends BaseDAO implements IDAO {
    + guardar(servicio: ServicioHotel): void
    + eliminar(id: String): boolean
    + buscarPorId(id: String): ServicioHotel
    + listarTodo(): List<ServicioHotel>
}

class FacturaDAO extends BaseDAO implements IDAO {
    + guardar(factura: FacturaInformativa): void
    + eliminar(id: String): boolean
    + buscarPorId(id: String): FacturaInformativa
    + listarTodo(): List<FacturaInformativa>
}

' --- CLASE CENTRAL: HOTEL ---
class Hotel {
    - idHotel: String
    - nombre: String
    - direccion: String
    - telefono: String
    - habitaciones: List<Habitacion>
    - clientes: List<Cliente>
    - reservas: List<Reserva>
    - serviciosDisponibles: List<ServicioHotel>
    + registrarHabitacion(h: Habitacion): void
    + eliminarHabitacion(num: int): void
    + registrarCliente(c: Cliente): void
    + buscarCliente(id: String): Cliente
    + listarClientesHabituales(): List<Cliente>
    + crearReserva(c: Cliente, habs: List<Habitacion>, fecha: String, dias: int): Reserva
    + cancelarReserva(id: String): void
    + getHabitacionesDisponibles(tipo: String): List<Habitacion>
    + registrarServicioHotel(s: ServicioHotel): void
    + generarFacturaInformativa(r: Reserva): FacturaInformativa
    + guardarDatos(): void
    + cargarDatos(): void
}

' --- RELACIONES ---
Hotel "1" *-- "many" Habitacion : administra >
Hotel "1" *-- "many" Cliente : gestiona >
Hotel "1" *-- "many" Reserva : controla >
Hotel "1" *-- "many" ServicioHotel : catálogo >

Hotel ..> ClienteDAO : -usa >
Hotel ..> ReservaDAO : -usa >
Hotel ..> HabitacionDAO : -usa >
Hotel ..> ServicioDAO : -usa >
Hotel ..> FacturaDAO : -usa >

Reserva "many" o-- "1" Cliente : pertenece a >
Reserva "1" *-- "many" Habitacion : incluye >
Reserva "1" *-- "many" ServicioConsumido : contiene >
ServicioConsumido "many" --> "1" ServicioHotel : referencia >
FacturaInformativa "1" -- "1" Reserva : se genera de >

Habitacion ..> EstadoHabitacion : usa
Reserva ..> EstadoReserva : usa
BaseDAO ..> MetodoPersistencia : define tipo
@enduml
