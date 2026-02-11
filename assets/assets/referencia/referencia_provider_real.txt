import 'dart:typed_data';
import 'package:file_picker/file_picker.dart';
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'package:pluto_grid/pluto_grid.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

import 'package:nethive_neo/helpers/globals.dart';
import 'package:nethive_neo/models/nethive/categoria_componente_model.dart';
import 'package:nethive_neo/models/nethive/componente_model.dart';
import 'package:nethive_neo/models/nethive/distribucion_model.dart';
import 'package:nethive_neo/models/nethive/conexion_componente_model.dart';
import 'package:nethive_neo/models/nethive/topologia_completa_model.dart';
import 'package:nethive_neo/models/nethive/rol_logico_componente_model.dart';
import 'package:nethive_neo/models/nethive/tipo_distribucion_model.dart';
import 'package:nethive_neo/models/nethive/detalle_cable_model.dart';
import 'package:nethive_neo/models/nethive/detalle_switch_model.dart';
import 'package:nethive_neo/models/nethive/detalle_patch_panel_model.dart';
import 'package:nethive_neo/models/nethive/detalle_rack_model.dart';
import 'package:nethive_neo/models/nethive/detalle_organizador_model.dart';
import 'package:nethive_neo/models/nethive/detalle_ups_model.dart';
import 'package:nethive_neo/models/nethive/detalle_router_firewall_model.dart';
import 'package:nethive_neo/models/nethive/detalle_equipo_activo_model.dart';
import 'package:nethive_neo/models/nethive/vista_conexiones_por_cables_model.dart';
import 'package:nethive_neo/models/nethive/vista_topologia_por_negocio_model.dart';
import 'package:nethive_neo/models/nethive/rack_con_componentes_model.dart';

class ComponentesProvider extends ChangeNotifier {
  // State managers
  PlutoGridStateManager? componentesStateManager;
  PlutoGridStateManager? categoriasStateManager;

  // Controladores de búsqueda
  final busquedaComponenteController = TextEditingController();
  final busquedaCategoriaController = TextEditingController();

  // Listas principales
  List<CategoriaComponente> categorias = [];
  List<RolLogicoComponente> rolesLogicos = [];
  List<TipoDistribucion> tiposDistribucion = [];
  List<Componente> componentes = [];
  List<PlutoRow> componentesRows = [];
  List<PlutoRow> categoriasRows = [];

  // Nueva estructura de topología optimizada
  TopologiaCompleta? topologiaCompleta;
  List<ComponenteTopologia> componentesTopologia = [];
  List<ConexionDatos> conexionesDatos = [];
  List<ConexionEnergia> conexionesEnergia = [];

  // Listas para retrocompatibilidad
  List<Distribucion> distribuciones = [];
  List<ConexionComponente> conexiones = [];
  List<VistaConexionesPorCables> conexionesConCables = [];
  List<VistaTopologiaPorNegocio> topologiaOptimizada = [];

  // Variables para formularios
  String? imagenFileName;
  Uint8List? imagenToUpload;
  String? negocioSeleccionadoId;
  String? negocioSeleccionadoNombre;
  String? empresaSeleccionadaId;
  int? categoriaSeleccionadaId;
  bool showDetallesEspecificos = false;

  // Variables para gestión de topología
  bool isLoadingTopologia = false;
  bool isLoadingRacks = false;
  List<String> problemasTopologia = [];

  // Detalles específicos por tipo de componente
  DetalleCable? detalleCable;
  DetalleSwitch? detalleSwitch;
  DetallePatchPanel? detallePatchPanel;
  DetalleRack? detalleRack;
  DetalleOrganizador? detalleOrganizador;
  DetalleUps? detalleUps;
  DetalleRouterFirewall? detalleRouterFirewall;
  DetalleEquipoActivo? detalleEquipoActivo;

  // Nueva lista para racks con componentes
  List<RackConComponentes> racksConComponentes = [];

  // Variable para controlar si el provider está activo
  bool _isDisposed = false;

  ComponentesProvider() {
    _inicializarDatos();
  }

  @override
  void dispose() {
    _isDisposed = true;
    busquedaComponenteController.dispose();
    busquedaCategoriaController.dispose();
    super.dispose();
  }

  // Método seguro para notificar listeners
  void _safeNotifyListeners() {
    if (!_isDisposed) {
      notifyListeners();
    }
  }

  // INICIALIZACIÓN
  Future<void> _inicializarDatos() async {
    try {
      await Future.wait([
        getCategorias(),
        getRolesLogicos(),
        getTiposDistribucion(),
      ]);
    } catch (e) {
      print('Error en inicialización: ${e.toString()}');
    }
  }

  // MÉTODOS PARA ROLES LÓGICOS
  Future<void> getRolesLogicos() async {
    try {
      final res = await supabaseLU
          .from('rol_logico_componente')
          .select()
          .order('nombre', ascending: true);

      rolesLogicos = (res as List<dynamic>)
          .map((rol) => RolLogicoComponente.fromMap(rol))
          .toList();

      _safeNotifyListeners();
    } catch (e) {
      print('Error en getRolesLogicos: ${e.toString()}');
    }
  }

  // MÉTODOS PARA TIPOS DE DISTRIBUCIÓN
  Future<void> getTiposDistribucion() async {
    try {
      final res = await supabaseLU
          .from('tipo_distribucion')
          .select()
          .order('nombre', ascending: true);

      tiposDistribucion = (res as List<dynamic>)
          .map((tipo) => TipoDistribucion.fromMap(tipo))
          .toList();

      _safeNotifyListeners();
    } catch (e) {
      print('Error en getTiposDistribucion: ${e.toString()}');
    }
  }

  // MÉTODO PRINCIPAL PARA OBTENER TOPOLOGÍA COMPLETA
  Future<void> getTopologiaPorNegocio(String negocioId) async {
    try {
      isLoadingTopologia = true;
      _safeNotifyListeners();

      print(
          'Llamando a función RPC fn_topologia_por_negocio con negocio_id: $negocioId');

      final response =
          await supabaseLU.rpc('fn_topologia_por_negocio', params: {
        'p_negocio_id': negocioId,
      }).select();

      /*   print('Respuesta RPC recibida: $response'); */

      if (response != null) {
        topologiaCompleta = TopologiaCompleta.fromJson(response);

        // Actualizar listas individuales
        componentesTopologia = topologiaCompleta!.componentes;
        conexionesDatos = topologiaCompleta!.conexionesDatos;
        conexionesEnergia = topologiaCompleta!.conexionesEnergia;

        // Sincronizar con estructuras anteriores para retrocompatibilidad
        _sincronizarEstructurasAnteriores();

        print('Topología cargada exitosamente:');
        print('- Componentes: ${componentesTopologia.length}');
        print('- Conexiones de datos: ${conexionesDatos.length}');
        print('- Conexiones de energía: ${conexionesEnergia.length}');

        problemasTopologia = _validarTopologiaCompleta();
      } else {
        print('Respuesta RPC nula, cargando datos con métodos alternativos');
        await _cargarTopologiaAlternativa(negocioId);
      }
    } catch (e) {
      print('Error en getTopologiaPorNegocio: ${e.toString()}');
      await _cargarTopologiaAlternativa(negocioId);
    } finally {
      isLoadingTopologia = false;
      _safeNotifyListeners();
    }
  }

  void _sincronizarEstructurasAnteriores() {
    // Convertir ComponenteTopologia a Componente para retrocompatibilidad
    componentes = componentesTopologia.map((ct) {
      return Componente(
        id: ct.id,
        negocioId: negocioSeleccionadoId ?? '',
        categoriaId: ct.categoriaId,
        nombre: ct.nombre,
        descripcion: ct.descripcion,
        ubicacion: ct.ubicacion,
        imagenUrl: ct.imagenUrl,
        rfid: ct.rfid, // ← NUEVO campo RFID
        enUso: ct.enUso,
        activo: ct.activo,
        fechaRegistro: ct.fechaRegistro,
        distribucionId: ct.distribucionId,
      );
    }).toList();

    // Convertir ConexionDatos a ConexionComponente para retrocompatibilidad
    conexiones = conexionesDatos.map((cd) {
      return ConexionComponente(
        id: cd.id,
        componenteOrigenId: cd.componenteOrigenId,
        componenteDestinoId: cd.componenteDestinoId,
        descripcion: cd.descripcion,
        activo: cd.activo,
      );
    }).toList();

    // Construir filas para la tabla
    _buildComponentesRows();
  }

  Future<void> _cargarTopologiaAlternativa(String negocioId) async {
    try {
      problemasTopologia = ['Usando método alternativo de carga de datos'];

      await Future.wait([
        getComponentesPorNegocio(negocioId),
        getDistribucionesPorNegocio(negocioId),
        getConexionesPorNegocio(negocioId),
      ]);

      problemasTopologia.addAll(validarTopologia());
    } catch (e) {
      problemasTopologia = [
        'Error al cargar datos de topología: ${e.toString()}'
      ];
    }
  }

  List<String> _validarTopologiaCompleta() {
    List<String> problemas = [];

    if (componentesTopologia.isEmpty) {
      problemas.add('No se encontraron componentes para este negocio');
      return problemas;
    }

    final mdfComponents = componentesTopologia.where((c) => c.esMDF).toList();
    final idfComponents = componentesTopologia.where((c) => c.esIDF).toList();

    if (mdfComponents.isEmpty) {
      problemas
          .add('No se encontraron componentes MDF (distribución principal)');
    }

    if (idfComponents.isEmpty) {
      problemas.add(
          'No se encontraron componentes IDF (distribuciones intermedias)');
    }

    final sinUbicacion = componentesTopologia
        .where((c) =>
            c.activo && (c.ubicacion == null || c.ubicacion!.trim().isEmpty))
        .length;
    if (sinUbicacion > 0) {
      problemas.add('$sinUbicacion componentes activos sin ubicación definida');
    }

    final componentesActivos =
        componentesTopologia.where((c) => c.activo).length;
    final conexionesDatosActivas =
        conexionesDatos.where((c) => c.activo).length;

    if (componentesActivos > 1 && conexionesDatosActivas == 0) {
      problemas.add('No se encontraron conexiones de datos entre componentes');
    }

    return problemas;
  }

  // MÉTODOS PARA CATEGORÍAS (mantenidos para retrocompatibilidad)
  Future<void> getCategorias([String? busqueda]) async {
    try {
      var query = supabaseLU.from('categoria_componente').select();

      if (busqueda != null && busqueda.isNotEmpty) {
        query = query.ilike('nombre', '%$busqueda%');
      }

      final res = await query.order('nombre', ascending: true);

      categorias = (res as List<dynamic>)
          .map((categoria) => CategoriaComponente.fromMap(categoria))
          .toList();
      print('Categorías cargadas: ${categorias[0].colorCategoria}');

      _buildCategoriasRows();
      _safeNotifyListeners();
    } catch (e) {
      print('Error en getCategorias: ${e.toString()}');
    }
  }

  void _buildCategoriasRows() {
    categoriasRows.clear();

    for (CategoriaComponente categoria in categorias) {
      categoriasRows.add(PlutoRow(cells: {
        'id': PlutoCell(value: categoria.id),
        'nombre': PlutoCell(value: categoria.nombre),
        'color_categoria': PlutoCell(value: categoria.colorCategoria ?? ''),
        'editar': PlutoCell(value: categoria.id),
        'eliminar': PlutoCell(value: categoria.id),
      }));
    }
  }

  // MÉTODOS PARA COMPONENTES (mantenidos para retrocompatibilidad)
  Future<void> getComponentesPorNegocio(String negocioId,
      [String? busqueda]) async {
    try {
      var query = supabaseLU.from('componente').select('''
            *,
            categoria_componente!inner(id, nombre)
          ''').eq('negocio_id', negocioId);

      if (busqueda != null && busqueda.isNotEmpty) {
        query = query.or(
            'nombre.ilike.%$busqueda%,descripcion.ilike.%$busqueda%,ubicacion.ilike.%$busqueda%,rfid.ilike.%$busqueda%');
      }

      final res = await query.order('fecha_registro', ascending: false);

      componentes = (res as List<dynamic>)
          .map((componente) => Componente.fromMap(componente))
          .toList();

      _buildComponentesRows();
      _safeNotifyListeners();
    } catch (e) {
      print('Error en getComponentesPorNegocio: ${e.toString()}');
    }
  }

  void _buildComponentesRows() {
    componentesRows.clear();

    for (Componente componente in componentes) {
      componentesRows.add(PlutoRow(cells: {
        'numero_fila': PlutoCell(value: ''), // Campo para numeración automática
        'id': PlutoCell(value: componente.id),
        'negocio_id': PlutoCell(value: componente.negocioId),
        'categoria_id': PlutoCell(value: componente.categoriaId),
        'categoria_nombre': PlutoCell(
            value: getCategoriaById(componente.categoriaId)?.nombre ??
                'Sin categoría'),
        'color_categoria': PlutoCell(
            value:
                getCategoriaById(componente.categoriaId)?.colorCategoria ?? ''),
        'nombre': PlutoCell(value: componente.nombre),
        'descripcion': PlutoCell(value: componente.descripcion ?? ''),
        'rfid': PlutoCell(value: componente.rfid ?? ''), // ← NUEVO campo RFID
        'en_uso': PlutoCell(value: componente.enUso ? 'Sí' : 'No'),
        'activo': PlutoCell(value: componente.activo ? 'Sí' : 'No'),
        'ubicacion': PlutoCell(value: componente.ubicacion ?? ''),
        'fecha_registro':
            PlutoCell(value: componente.fechaRegistro.toString().split(' ')[0]),
        'imagen_url': PlutoCell(
          value: componente.imagenUrl != null
              ? "${supabaseLU.supabaseUrl}/storage/v1/object/public/nethive/componentes/${componente.imagenUrl}?${DateTime.now().millisecondsSinceEpoch}"
              : '',
        ),
        'editar': PlutoCell(value: componente.id),
        'eliminar': PlutoCell(value: componente.id),
        'ver_detalles': PlutoCell(value: componente.id),
      }));
    }
  }

  // MÉTODOS PARA DISTRIBUCIONES (mantenidos para retrocompatibilidad)
  Future<void> getDistribucionesPorNegocio(String negocioId) async {
    try {
      final res = await supabaseLU
          .from('distribucion')
          .select()
          .eq('negocio_id', negocioId)
          .order('tipo', ascending: false);

      distribuciones = (res as List<dynamic>)
          .map((distribucion) => Distribucion.fromMap(distribucion))
          .toList();

      print('Distribuciones cargadas: ${distribuciones.length}');
    } catch (e) {
      print('Error en getDistribucionesPorNegocio: ${e.toString()}');
      distribuciones = [];
    }
  }

  // MÉTODOS PARA CONEXIONES (mantenidos para retrocompatibilidad)
  Future<void> getConexionesPorNegocio(String negocioId) async {
    try {
      List<dynamic> res;

      try {
        res = await supabaseLU
            .from('vista_conexiones_por_negocio')
            .select()
            .eq('negocio_id', negocioId);
      } catch (e) {
        final componentesDelNegocio = await supabaseLU
            .from('componente')
            .select('id')
            .eq('negocio_id', negocioId);

        if (componentesDelNegocio.isEmpty) {
          conexiones = [];
          return;
        }

        final componenteIds =
            componentesDelNegocio.map((comp) => comp['id'] as String).toList();

        res = await supabaseLU
            .from('conexion_componente')
            .select()
            .or('componente_origen_id.in.(${componenteIds.join(',')}),componente_destino_id.in.(${componenteIds.join(',')})')
            .eq('activo', true);
      }

      conexiones = (res)
          .map((conexion) => ConexionComponente.fromMap(conexion))
          .toList();

      print('Conexiones cargadas: ${conexiones.length}');
    } catch (e) {
      print('Error en getConexionesPorNegocio: ${e.toString()}');
      conexiones = [];
    }
  }

  // MÉTODOS DE VALIDACIÓN (mantenidos para retrocompatibilidad)
  List<String> validarTopologia() {
    List<String> problemas = [];

    if (componentes.isEmpty) {
      problemas.add('No se encontraron componentes para este negocio');
      return problemas;
    }

    final mdfComponents = getComponentesPorTipo('mdf');
    final idfComponents = getComponentesPorTipo('idf');

    if (mdfComponents.isEmpty) {
      problemas
          .add('No se encontraron componentes MDF (distribución principal)');
    }

    if (idfComponents.isEmpty) {
      problemas.add(
          'No se encontraron componentes IDF (distribuciones intermedias)');
    }

    final sinUbicacion = componentes
        .where((c) =>
            c.activo && (c.ubicacion == null || c.ubicacion!.trim().isEmpty))
        .length;
    if (sinUbicacion > 0) {
      problemas.add('$sinUbicacion componentes activos sin ubicación definida');
    }

    final componentesActivos = componentes.where((c) => c.activo).length;
    final conexionesActivas = conexiones.where((c) => c.activo).length;

    if (componentesActivos > 1 && conexionesActivas == 0) {
      problemas.add('No se encontraron conexiones entre componentes');
    }

    return problemas;
  }

  // MÉTODOS DE UTILIDAD (mantenidos)
  CategoriaComponente? getCategoriaById(int categoriaId) {
    try {
      return categorias.firstWhere((c) => c.id == categoriaId);
    } catch (e) {
      return null;
    }
  }

  Componente? getComponenteById(String componenteId) {
    try {
      return componentes.firstWhere((c) => c.id == componenteId);
    } catch (e) {
      return null;
    }
  }

  ComponenteTopologia? getComponenteTopologiaById(String componenteId) {
    try {
      return componentesTopologia.firstWhere((c) => c.id == componenteId);
    } catch (e) {
      return null;
    }
  }

  List<Componente> getComponentesPorTipo(String tipo) {
    return componentes.where((c) {
      if (!c.activo) return false;

      final categoria = getCategoriaById(c.categoriaId);
      final nombreCategoria = categoria?.nombre.toLowerCase() ?? '';

      switch (tipo.toLowerCase()) {
        case 'mdf':
          return c.ubicacion?.toLowerCase().contains('mdf') == true ||
              nombreCategoria.contains('mdf') ||
              c.descripcion?.toLowerCase().contains('mdf') == true;
        case 'idf':
          return c.ubicacion?.toLowerCase().contains('idf') == true ||
              nombreCategoria.contains('idf') ||
              c.descripcion?.toLowerCase().contains('idf') == true;
        case 'switch':
          return nombreCategoria.contains('switch');
        case 'router':
          return nombreCategoria.contains('router') ||
              nombreCategoria.contains('firewall');
        case 'servidor':
        case 'server':
          return nombreCategoria.contains('servidor') ||
              nombreCategoria.contains('server');
        default:
          return false;
      }
    }).toList();
  }

  // MÉTODOS PARA IMÁGENES (mantenidos)
  Future<void> selectImagen() async {
    imagenFileName = null;
    imagenToUpload = null;

    FilePickerResult? picker = await FilePicker.platform.pickFiles(
      type: FileType.custom,
      allowedExtensions: ['jpg', 'png', 'jpeg'],
    );

    if (picker != null) {
      var now = DateTime.now();
      var formatter = DateFormat('yyyyMMddHHmmss');
      var timestamp = formatter.format(now);

      imagenFileName = 'componente-$timestamp-${picker.files.single.name}';
      imagenToUpload = picker.files.single.bytes;
    }

    _safeNotifyListeners();
  }

  Future<String?> uploadImagen() async {
    if (imagenToUpload != null && imagenFileName != null) {
      await supabaseLU.storage.from('nethive/componentes').uploadBinary(
            imagenFileName!,
            imagenToUpload!,
            fileOptions: const FileOptions(
              cacheControl: '3600',
              upsert: false,
            ),
          );
      return imagenFileName;
    }
    return null;
  }

  // MÉTODOS DE UTILIDAD PARA FORMULARIOS (mantenidos)
  void resetFormData() {
    imagenFileName = null;
    imagenToUpload = null;
    categoriaSeleccionadaId = null;
    _safeNotifyListeners();
  }

  void buscarComponentes(String busqueda) {
    if (negocioSeleccionadoId != null) {
      getComponentesPorNegocio(
          negocioSeleccionadoId!, busqueda.isEmpty ? null : busqueda);
    }
  }

  void buscarCategorias(String busqueda) {
    getCategorias(busqueda.isEmpty ? null : busqueda);
  }

  Widget? getImageWidget(dynamic image,
      {double height = 100, double width = 100}) {
    if (image == null || image.toString().isEmpty) {
      return Container(
        height: height,
        width: width,
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(4),
        ),
        child: Image.asset(
          'assets/images/placeholder_no_image.jpg',
          height: height,
          width: width,
          fit: BoxFit.cover,
        ),
      );
    } else if (image is Uint8List) {
      return Image.memory(
        image,
        height: height,
        width: width,
        fit: BoxFit.cover,
        errorBuilder: (context, error, stackTrace) {
          return Image.asset(
            'assets/images/placeholder_no_image.jpg',
            height: height,
            width: width,
            fit: BoxFit.cover,
          );
        },
      );
    } else if (image is String) {
      return Image.network(
        image,
        height: height,
        width: width,
        fit: BoxFit.cover,
        errorBuilder: (context, error, stackTrace) {
          return Image.asset(
            'assets/images/placeholder_no_image.jpg',
            height: height,
            width: width,
            fit: BoxFit.cover,
          );
        },
      );
    }
    return Image.asset(
      'assets/images/placeholder_no_image.jpg',
      height: height,
      width: width,
      fit: BoxFit.cover,
    );
  }

  // MÉTODOS CRUD MANTENIDOS PARA COMPATIBILIDAD
  Future<bool> crearComponente({
    required String negocioId,
    required int categoriaId,
    required String nombre,
    String? descripcion,
    required bool enUso,
    required bool activo,
    String? ubicacion,
    String? rfid, // ← NUEVO parámetro
  }) async {
    try {
      final imagenUrl = await uploadImagen();

      final res = await supabaseLU.from('componente').insert({
        'negocio_id': negocioId,
        'categoria_id': categoriaId,
        'nombre': nombre,
        'descripcion': descripcion,
        'en_uso': enUso,
        'activo': activo,
        'ubicacion': ubicacion,
        'imagen_url': imagenUrl,
        'rfid': rfid, // ← NUEVO campo en la inserción
      }).select();

      if (res.isNotEmpty) {
        await getTopologiaPorNegocio(negocioId);
        resetFormData();
        return true;
      }
      return false;
    } catch (e) {
      print('Error en crearComponente: ${e.toString()}');
      return false;
    }
  }

  Future<bool> actualizarComponente({
    required String componenteId,
    required String negocioId,
    required int categoriaId,
    required String nombre,
    String? descripcion,
    required bool enUso,
    required bool activo,
    String? ubicacion,
    String? rfid, // ← NUEVO parámetro
    bool actualizarImagen = false,
  }) async {
    try {
      Map<String, dynamic> updateData = {
        'categoria_id': categoriaId,
        'nombre': nombre,
        'descripcion': descripcion,
        'en_uso': enUso,
        'activo': activo,
        'ubicacion': ubicacion,
        'rfid': rfid, // ← NUEVO campo en la actualización
      };

      if (actualizarImagen) {
        final imagenUrl = await uploadImagen();
        if (imagenUrl != null) {
          updateData['imagen_url'] = imagenUrl;
        }
      }

      final res = await supabaseLU
          .from('componente')
          .update(updateData)
          .eq('id', componenteId)
          .select();

      if (res.isNotEmpty) {
        await getTopologiaPorNegocio(negocioId);
        resetFormData();
        return true;
      }
      return false;
    } catch (e) {
      print('Error en actualizarComponente: ${e.toString()}');
      return false;
    }
  }

  Future<bool> eliminarComponente(String componenteId) async {
    try {
      final componenteData = await supabaseLU
          .from('componente')
          .select('imagen_url')
          .eq('id', componenteId)
          .maybeSingle();

      String? imagenUrl;
      if (componenteData != null && componenteData['imagen_url'] != null) {
        imagenUrl = componenteData['imagen_url'] as String;
      }

      await _eliminarDetallesComponente(componenteId);
      await supabaseLU.from('componente').delete().eq('id', componenteId);

      if (!_isDisposed && negocioSeleccionadoId != null) {
        await getTopologiaPorNegocio(negocioSeleccionadoId!);
      }

      if (imagenUrl != null) {
        try {
          await supabaseLU.storage
              .from('nethive')
              .remove(["componentes/$imagenUrl"]);
        } catch (storageError) {
          print(
              'Error al eliminar imagen del storage: ${storageError.toString()}');
        }
      }

      return true;
    } catch (e) {
      print('Error en eliminarComponente: ${e.toString()}');
      return false;
    }
  }

  Future<void> _eliminarDetallesComponente(String componenteId) async {
    try {
      await Future.wait([
        supabaseLU
            .from('detalle_cable')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU
            .from('detalle_switch')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU
            .from('detalle_patch_panel')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU
            .from('detalle_rack')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU
            .from('detalle_organizador')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU
            .from('detalle_ups')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU
            .from('detalle_router_firewall')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU
            .from('detalle_equipo_activo')
            .delete()
            .eq('componente_id', componenteId),
        supabaseLU.from('conexion_componente').delete().or(
            'componente_origen_id.eq.$componenteId,componente_destino_id.eq.$componenteId'),
      ]);
    } catch (e) {
      print('Error al eliminar detalles del componente: ${e.toString()}');
    }
  }

  Future<bool> crearCategoria(String nombre) async {
    try {
      final res = await supabaseLU.from('categoria_componente').insert({
        'nombre': nombre,
      }).select();

      if (res.isNotEmpty) {
        await getCategorias();
        return true;
      }
      return false;
    } catch (e) {
      print('Error en crearCategoria: ${e.toString()}');
      return false;
    }
  }

  Future<bool> actualizarCategoria(int id, String nombre) async {
    try {
      final res = await supabaseLU
          .from('categoria_componente')
          .update({'nombre': nombre})
          .eq('id', id)
          .select();

      if (res.isNotEmpty) {
        await getCategorias();
        return true;
      }
      return false;
    } catch (e) {
      print('Error en actualizarCategoria: ${e.toString()}');
      return false;
    }
  }

  Future<bool> eliminarCategoria(int id) async {
    try {
      await supabaseLU.from('categoria_componente').delete().eq('id', id);
      await getCategorias();
      return true;
    } catch (e) {
      print('Error en eliminarCategoria: ${e.toString()}');
      return false;
    }
  }

  // MÉTODOS DE GESTIÓN DE TOPOLOGÍA OPTIMIZADA
  Future<void> setNegocioSeleccionado(
      String negocioId, String negocioNombre, String empresaId) async {
    try {
      negocioSeleccionadoId = negocioId;
      negocioSeleccionadoNombre = negocioNombre;
      empresaSeleccionadaId = empresaId;

      _limpiarDatosAnteriores();

      // Cargar datos en paralelo
      await Future.wait([
        getTopologiaPorNegocio(negocioId),
        getRacksConComponentes(negocioId),
      ]);

      _safeNotifyListeners();
    } catch (e) {
      print('Error en setNegocioSeleccionado: ${e.toString()}');
    }
  }

  void _limpiarDatosAnteriores() {
    topologiaCompleta = null;
    componentesTopologia.clear();
    conexionesDatos.clear();
    conexionesEnergia.clear();

    componentes.clear();
    distribuciones.clear();
    conexiones.clear();
    conexionesConCables.clear();
    topologiaOptimizada.clear();
    componentesRows.clear();
    showDetallesEspecificos = false;

    detalleCable = null;
    detalleSwitch = null;
    detallePatchPanel = null;
    detalleRack = null;
    detalleOrganizador = null;
    detalleUps = null;
    detalleRouterFirewall = null;
    detalleEquipoActivo = null;

    racksConComponentes.clear();
    isLoadingRacks = false;
  }

  // MÉTODOS DE UTILIDAD PARA TOPOLOGÍA
  List<ComponenteTopologia> getComponentesPorTipoTopologia(String tipo) {
    return componentesTopologia.where((c) {
      if (!c.activo) return false;

      switch (tipo.toLowerCase()) {
        case 'mdf':
          return c.esMDF;
        case 'idf':
          return c.esIDF;
        case 'switch':
          return c.esSwitch;
        case 'router':
          return c.esRouter;
        case 'servidor':
        case 'server':
          return c.esServidor;
        case 'ups':
          return c.esUPS;
        case 'rack':
          return c.esRack;
        case 'patch':
        case 'panel':
          return c.esPatchPanel;
        default:
          return false;
      }
    }).toList()
      ..sort((a, b) => a.prioridadTopologia.compareTo(b.prioridadTopologia));
  }

  List<ComponenteTopologia> getComponentesMDF() {
    return getComponentesPorTipoTopologia('mdf');
  }

  List<ComponenteTopologia> getComponentesIDF() {
    return getComponentesPorTipoTopologia('idf');
  }

  List<ComponenteTopologia> getComponentesSwitch() {
    return getComponentesPorTipoTopologia('switch');
  }

  List<ComponenteTopologia> getComponentesRouter() {
    return getComponentesPorTipoTopologia('router');
  }

  List<ComponenteTopologia> getComponentesServidor() {
    return getComponentesPorTipoTopologia('servidor');
  }

  List<ConexionDatos> getConexionesPorComponente(String componenteId) {
    return conexionesDatos
        .where((c) =>
            c.componenteOrigenId == componenteId ||
            c.componenteDestinoId == componenteId)
        .toList();
  }

  List<ConexionEnergia> getConexionesEnergiaPorComponente(String componenteId) {
    return conexionesEnergia
        .where((c) => c.origenId == componenteId || c.destinoId == componenteId)
        .toList();
  }

  // MÉTODOS PARA CARGAR RACKS CON COMPONENTES
  Future<void> getRacksConComponentes(String negocioId) async {
    try {
      isLoadingRacks = true;
      _safeNotifyListeners();

      print(
          'Llamando a función RPC fn_racks_con_componentes con negocio_id: $negocioId');

      final response =
          await supabaseLU.rpc('fn_racks_con_componentes', params: {
        'p_negocio_id': negocioId,
      });

      print('Respuesta RPC racks: $response');

      if (response != null && response is List) {
        racksConComponentes =
            (response).map((rack) => RackConComponentes.fromMap(rack)).toList();

        print('Racks cargados: ${racksConComponentes.length}');
        for (var rack in racksConComponentes) {
          print(
              '- ${rack.nombreRack}: ${rack.cantidadComponentes} componentes');
        }
      } else {
        racksConComponentes = [];
        print('No se encontraron racks o respuesta vacía');
      }
    } catch (e) {
      print('Error en getRacksConComponentes: ${e.toString()}');
      racksConComponentes = [];
    } finally {
      isLoadingRacks = false;
      _safeNotifyListeners();
    }
  }

  // MÉTODOS DE UTILIDAD PARA RACKS
  RackConComponentes? getRackById(String rackId) {
    try {
      return racksConComponentes.firstWhere((rack) => rack.rackId == rackId);
    } catch (e) {
      return null;
    }
  }

  int get totalRacks => racksConComponentes.length;

  int get totalComponentesEnRacks => racksConComponentes.fold(
      0, (sum, rack) => sum + rack.cantidadComponentes);

  int get racksConComponentesActivos =>
      racksConComponentes.where((rack) => rack.componentesActivos > 0).length;

  double get porcentajeOcupacionPromedio {
    if (racksConComponentes.isEmpty) return 0.0;

    final totalOcupacion = racksConComponentes.fold(
        0.0, (sum, rack) => sum + rack.porcentajeOcupacion);

    return totalOcupacion / racksConComponentes.length;
  }

  List<RackConComponentes> get racksOrdenadosPorOcupacion {
    final racks = [...racksConComponentes];
    racks
        .sort((a, b) => b.porcentajeOcupacion.compareTo(a.porcentajeOcupacion));
    return racks;
  }

  List<RackConComponentes> get racksConProblemas {
    return racksConComponentes.where((rack) {
      // Rack con problemas si tiene componentes inactivos o sin posición U
      final componentesInactivos =
          rack.componentes.where((c) => !c.activo).length;

      final componentesSinPosicion =
          rack.componentes.where((c) => c.posicionU == null).length;

      return componentesInactivos > 0 || componentesSinPosicion > 0;
    }).toList();
  }
}
