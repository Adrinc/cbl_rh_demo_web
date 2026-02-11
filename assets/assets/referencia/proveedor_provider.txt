import 'package:flutter/material.dart';
import 'package:facturacion_demo/models/models.dart';
import 'package:facturacion_demo/data/mock_data.dart';
import 'package:facturacion_demo/helpers/constants.dart';

/// ============================================================================
/// PROVEEDOR PROVIDER
/// ============================================================================
/// Gestiona el estado y operaciones CRUD de proveedores (en memoria)
/// Los cambios persisten durante la sesión pero se pierden al reiniciar
/// ============================================================================
class ProveedorProvider extends ChangeNotifier {
  List<Proveedor> _proveedores = [];

  List<Proveedor> get proveedores => List.unmodifiable(_proveedores);

  /// Constructor que carga los datos mock
  ProveedorProvider() {
    _loadMockData();
  }

  /// Carga los datos mock iniciales
  void _loadMockData() {
    _proveedores = List.from(mockProveedores);
    notifyListeners();
  }

  /// Obtiene un proveedor por ID
  Proveedor? getProveedorById(String id) {
    try {
      return _proveedores.firstWhere((p) => p.id == id);
    } catch (e) {
      return null;
    }
  }

  /// Obtiene proveedores activos
  List<Proveedor> getProveedoresActivos() {
    return _proveedores
        .where((p) => p.estado == EstadoProveedor.activo)
        .toList();
  }

  /// Obtiene proveedores por esquema
  List<Proveedor> getProveedoresByEsquema(String esquema) {
    return _proveedores.where((p) => p.esquemaActivo == esquema).toList();
  }

  /// Obtiene proveedores con DPP permitido
  List<Proveedor> getProveedoresConDPP() {
    return _proveedores.where((p) => p.dppPermitido).toList();
  }

  /// Agrega un nuevo proveedor
  void addProveedor(Proveedor proveedor) {
    _proveedores.add(proveedor);
    notifyListeners();
  }

  /// Actualiza un proveedor existente
  void updateProveedor(Proveedor proveedor) {
    final index = _proveedores.indexWhere((p) => p.id == proveedor.id);
    if (index != -1) {
      _proveedores[index] = proveedor;
      notifyListeners();
    }
  }

  /// Elimina un proveedor por ID
  void deleteProveedor(String id) {
    _proveedores.removeWhere((p) => p.id == id);
    notifyListeners();
  }

  /// Activa/Desactiva un proveedor
  void toggleEstadoProveedor(String id) {
    final index = _proveedores.indexWhere((p) => p.id == id);
    if (index != -1) {
      final proveedor = _proveedores[index];
      _proveedores[index] = Proveedor(
        id: proveedor.id,
        nombre: proveedor.nombre,
        rfc: proveedor.rfc,
        esquemaActivo: proveedor.esquemaActivo,
        diasPago: proveedor.diasPago,
        dppPermitido: proveedor.dppPermitido,
        porcentajeDPPBase: proveedor.porcentajeDPPBase,
        diasGraciaDPP: proveedor.diasGraciaDPP,
        contacto: proveedor.contacto,
        email: proveedor.email,
        estado: proveedor.estado == EstadoProveedor.activo
            ? EstadoProveedor.inactivo
            : EstadoProveedor.activo,
      );
      notifyListeners();
    }
  }

  /// Resetea los datos a los valores mock originales
  void resetData() {
    _loadMockData();
  }

  /// Obtiene el conteo de proveedores por estado
  Map<String, int> getProveedoresCountByEstado() {
    return {
      EstadoProveedor.activo:
          _proveedores.where((p) => p.estado == EstadoProveedor.activo).length,
      EstadoProveedor.inactivo: _proveedores
          .where((p) => p.estado == EstadoProveedor.inactivo)
          .length,
    };
  }

  /// Obtiene el conteo de proveedores por esquema
  Map<String, int> getProveedoresCountByEsquema() {
    return {
      EsquemaPago.pull:
          _proveedores.where((p) => p.esquemaActivo == EsquemaPago.pull).length,
      EsquemaPago.push:
          _proveedores.where((p) => p.esquemaActivo == EsquemaPago.push).length,
    };
  }
}
