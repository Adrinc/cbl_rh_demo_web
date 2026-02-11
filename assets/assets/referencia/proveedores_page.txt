import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:facturacion_demo/providers/proveedor_provider.dart';
import 'package:facturacion_demo/models/models.dart';
import 'package:facturacion_demo/theme/theme.dart';
import 'package:facturacion_demo/helpers/constants.dart';
import 'package:facturacion_demo/pages/proveedores/widgets/proveedor_pluto_grid.dart';
import 'package:facturacion_demo/pages/proveedores/widgets/proveedor_form.dart';
import 'package:facturacion_demo/functions/proveedor_logo.dart';

/// ============================================================================
/// PROVEEDORES PAGE
/// ============================================================================
/// Gestión de proveedores y acuerdos comerciales
/// CRUD completo con PlutoGrid en desktop y cards en móvil
/// ============================================================================
class ProveedoresPage extends StatefulWidget {
  const ProveedoresPage({super.key});

  @override
  State<ProveedoresPage> createState() => _ProveedoresPageState();
}

class _ProveedoresPageState extends State<ProveedoresPage> {
  String _filterEsquema = 'todos';
  String _filterDPP = 'todos';
  String _filterEstado = 'activo';
  String _searchQuery = '';

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context).extension<AppTheme>()!;
    final proveedorProvider = context.watch<ProveedorProvider>();
    final isMobile = MediaQuery.of(context).size.width <= mobileSize;

    // Filtrar proveedores
    List<Proveedor> proveedoresFiltrados =
        proveedorProvider.proveedores.where((p) {
      // Filtro de esquema
      if (_filterEsquema != 'todos' && p.esquemaActivo != _filterEsquema) {
        return false;
      }
      // Filtro de DPP
      if (_filterDPP == 'con_dpp' && !p.dppPermitido) {
        return false;
      }
      if (_filterDPP == 'sin_dpp' && p.dppPermitido) {
        return false;
      }
      // Filtro de estado
      if (_filterEstado != 'todos' &&
          p.estado.toString().split('.').last != _filterEstado) {
        return false;
      }
      // Búsqueda por texto
      if (_searchQuery.isNotEmpty) {
        final query = _searchQuery.toLowerCase();
        return p.nombre.toLowerCase().contains(query) ||
            p.rfc.toLowerCase().contains(query) ||
            p.contacto.toLowerCase().contains(query);
      }
      return true;
    }).toList();

    return Scaffold(
      backgroundColor: theme.primaryBackground,
      body: Column(
        children: [
          // Filtros superiores
          _buildFilters(theme, isMobile),
          const SizedBox(height: 16),

          // Tabla o cards
          Expanded(
            child: isMobile
                ? _buildMobileCards(proveedoresFiltrados, theme)
                : _buildDesktopTable(proveedoresFiltrados, theme),
          ),
        ],
      ),
      floatingActionButton: isMobile
          ? FloatingActionButton.extended(
              onPressed: () => _showProveedorForm(context, null),
              icon: const Icon(Icons.add),
              label: const Text('Nuevo Proveedor'),
              backgroundColor: theme.primary,
            )
          : null,
    );
  }

  Widget _buildFilters(AppTheme theme, bool isMobile) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: theme.surface,
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: theme.border, width: 1),
        boxShadow: [
          BoxShadow(
            color: theme.textPrimary.withOpacity(0.05),
            blurRadius: 10,
            offset: const Offset(0, 4),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            children: [
              Icon(Icons.business, color: theme.primary, size: 20),
              const SizedBox(width: 8),
              Text(
                'Filtros de Proveedores',
                style: TextStyle(
                  color: theme.textPrimary,
                  fontSize: 16,
                  fontWeight: FontWeight.w600,
                ),
              ),
              const Spacer(),
              if (!isMobile)
                ElevatedButton.icon(
                  onPressed: () => _showProveedorForm(context, null),
                  icon: const Icon(Icons.add, size: 18),
                  label: const Text('Nuevo Proveedor'),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: theme.primary,
                    foregroundColor: Colors.white,
                    padding: const EdgeInsets.symmetric(
                        horizontal: 16, vertical: 12),
                  ),
                ),
            ],
          ),
          const SizedBox(height: 16),

          // Filtros en fila (desktop) o columna (mobile)
          if (!isMobile)
            Row(
              children: [
                // Búsqueda
                Expanded(
                  flex: 2,
                  child: TextField(
                    onChanged: (value) => setState(() => _searchQuery = value),
                    decoration: InputDecoration(
                      hintText: 'Buscar proveedor...',
                      prefixIcon:
                          Icon(Icons.search, color: theme.textSecondary),
                      border: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(8),
                        borderSide: BorderSide(color: theme.border),
                      ),
                      enabledBorder: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(8),
                        borderSide: BorderSide(color: theme.border),
                      ),
                      focusedBorder: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(8),
                        borderSide: BorderSide(color: theme.primary, width: 2),
                      ),
                      filled: true,
                      fillColor: theme.primaryBackground,
                      contentPadding: const EdgeInsets.symmetric(
                          horizontal: 16, vertical: 14),
                    ),
                  ),
                ),
                const SizedBox(width: 12),
                // Filtro de Esquema
                Expanded(
                  child: _buildDropdownFilter(
                    'Esquema',
                    _filterEsquema,
                    [
                      {'value': 'todos', 'label': 'Todos'},
                      {'value': EsquemaPago.push, 'label': 'PUSH'},
                      {'value': EsquemaPago.pull, 'label': 'PULL'},
                    ],
                    (value) => setState(() => _filterEsquema = value!),
                    theme,
                    isMobile,
                  ),
                ),
                const SizedBox(width: 12),
                // Filtro de DPP
                Expanded(
                  child: _buildDropdownFilter(
                    'DPP',
                    _filterDPP,
                    [
                      {'value': 'todos', 'label': 'Todos'},
                      {'value': 'con_dpp', 'label': 'Con DPP'},
                      {'value': 'sin_dpp', 'label': 'Sin DPP'},
                    ],
                    (value) => setState(() => _filterDPP = value!),
                    theme,
                    isMobile,
                  ),
                ),
                const SizedBox(width: 12),
                // Filtro de Estado
                Expanded(
                  child: _buildDropdownFilter(
                    'Estado',
                    _filterEstado,
                    [
                      {'value': 'todos', 'label': 'Todos'},
                      {'value': 'activo', 'label': 'Activos'},
                      {'value': 'inactivo', 'label': 'Inactivos'},
                    ],
                    (value) => setState(() => _filterEstado = value!),
                    theme,
                    isMobile,
                  ),
                ),
              ],
            ),

          // Móvil: Filtros en columna
          if (isMobile) ...[
            TextField(
              onChanged: (value) => setState(() => _searchQuery = value),
              decoration: InputDecoration(
                hintText: 'Buscar proveedor...',
                prefixIcon: Icon(Icons.search, color: theme.textSecondary),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(8),
                ),
                filled: true,
                fillColor: theme.primaryBackground,
                contentPadding:
                    const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
              ),
            ),
            const SizedBox(height: 12),
            Row(
              children: [
                Expanded(
                  child: _buildDropdownFilter(
                      'Esquema',
                      _filterEsquema,
                      [
                        {'value': 'todos', 'label': 'Todos'},
                        {'value': EsquemaPago.push, 'label': 'PUSH'},
                        {'value': EsquemaPago.pull, 'label': 'PULL'},
                      ],
                      (value) => setState(() => _filterEsquema = value!),
                      theme,
                      isMobile),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: _buildDropdownFilter(
                      'DPP',
                      _filterDPP,
                      [
                        {'value': 'todos', 'label': 'Todos'},
                        {'value': 'con_dpp', 'label': 'Con DPP'},
                        {'value': 'sin_dpp', 'label': 'Sin DPP'},
                      ],
                      (value) => setState(() => _filterDPP = value!),
                      theme,
                      isMobile),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: _buildDropdownFilter(
                      'Estado',
                      _filterEstado,
                      [
                        {'value': 'todos', 'label': 'Todos'},
                        {'value': 'activo', 'label': 'Activos'},
                        {'value': 'inactivo', 'label': 'Inactivos'},
                      ],
                      (value) => setState(() => _filterEstado = value!),
                      theme,
                      isMobile),
                ),
              ],
            ),
          ],
        ],
      ),
    );
  }

  Widget _buildDropdownFilter(
    String label,
    String value,
    List<Map<String, String>> options,
    ValueChanged<String?> onChanged,
    AppTheme theme,
    bool isMobile,
  ) {
    return SizedBox(
      width: isMobile ? double.infinity : 180,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            label,
            style: TextStyle(
              color: theme.textSecondary,
              fontSize: 12,
              fontWeight: FontWeight.w500,
            ),
          ),
          const SizedBox(height: 4),
          DropdownButtonFormField<String>(
            value: value,
            onChanged: onChanged,
            decoration: InputDecoration(
              border: OutlineInputBorder(
                borderRadius: BorderRadius.circular(8),
                borderSide: BorderSide(color: theme.border),
              ),
              enabledBorder: OutlineInputBorder(
                borderRadius: BorderRadius.circular(8),
                borderSide: BorderSide(color: theme.border),
              ),
              focusedBorder: OutlineInputBorder(
                borderRadius: BorderRadius.circular(8),
                borderSide: BorderSide(color: theme.primary, width: 2),
              ),
              filled: true,
              fillColor: theme.primaryBackground,
              contentPadding:
                  const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
            ),
            items: options.map((option) {
              return DropdownMenuItem(
                value: option['value'],
                child: Text(
                  option['label']!,
                  style: TextStyle(color: theme.textPrimary, fontSize: 14),
                ),
              );
            }).toList(),
          ),
        ],
      ),
    );
  }

  Widget _buildDesktopTable(List<Proveedor> proveedores, AppTheme theme) {
    return Container(
      margin: const EdgeInsets.only(bottom: 16),
      child: ProveedorPlutoGrid(
        proveedores: proveedores,
        onEdit: (proveedor) => _showProveedorForm(context, proveedor),
        onView: (proveedor) => _showProveedorDetails(context, proveedor),
      ),
    );
  }

  Widget _buildMobileCards(List<Proveedor> proveedores, AppTheme theme) {
    if (proveedores.isEmpty) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.business_outlined, size: 64, color: theme.textSecondary),
            const SizedBox(height: 16),
            Text(
              'No hay proveedores',
              style: TextStyle(
                color: theme.textSecondary,
                fontSize: 16,
                fontWeight: FontWeight.w500,
              ),
            ),
          ],
        ),
      );
    }

    return ListView.builder(
      padding: const EdgeInsets.only(bottom: 80),
      itemCount: proveedores.length,
      itemBuilder: (context, index) {
        final proveedor = proveedores[index];
        final isActivo = proveedor.estado == EstadoProveedor.activo;

        return Container(
          margin: const EdgeInsets.only(bottom: 12),
          decoration: BoxDecoration(
            color: theme.surface,
            borderRadius: BorderRadius.circular(12),
            border: Border.all(color: theme.border, width: 1),
            boxShadow: [
              BoxShadow(
                color: theme.textPrimary.withOpacity(0.05),
                blurRadius: 8,
                offset: const Offset(0, 2),
              ),
            ],
          ),
          child: InkWell(
            onTap: () => _showProveedorDetails(context, proveedor),
            borderRadius: BorderRadius.circular(12),
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    children: [
                      Container(
                        width: 56,
                        height: 56,
                        decoration: BoxDecoration(
                          color: theme.primaryBackground,
                          borderRadius: BorderRadius.circular(12),
                          border: Border.all(
                            color: theme.border.withOpacity(0.3),
                            width: 1,
                          ),
                        ),
                        child: ClipRRect(
                          borderRadius: BorderRadius.circular(8),
                          child: Image.asset(
                            getProveedorLogoPath(proveedor.nombre),
                            fit: BoxFit.fill,
                            errorBuilder: (context, error, stackTrace) {
                              return Center(
                                child: Text(
                                  proveedor.nombre
                                      .substring(0, 1)
                                      .toUpperCase(),
                                  style: TextStyle(
                                    color: theme.primary,
                                    fontWeight: FontWeight.w600,
                                    fontSize: 20,
                                  ),
                                ),
                              );
                            },
                          ),
                        ),
                      ),
                      const SizedBox(width: 12),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              proveedor.nombre,
                              style: TextStyle(
                                color: theme.textPrimary,
                                fontSize: 16,
                                fontWeight: FontWeight.w600,
                              ),
                            ),
                            Text(
                              proveedor.rfc,
                              style: TextStyle(
                                color: theme.textSecondary,
                                fontSize: 12,
                              ),
                            ),
                          ],
                        ),
                      ),
                      Container(
                        padding: const EdgeInsets.symmetric(
                            horizontal: 12, vertical: 6),
                        decoration: BoxDecoration(
                          color: isActivo
                              ? theme.success.withOpacity(0.15)
                              : theme.textSecondary.withOpacity(0.15),
                          borderRadius: BorderRadius.circular(20),
                        ),
                        child: Text(
                          isActivo ? 'ACTIVO' : 'INACTIVO',
                          style: TextStyle(
                            color:
                                isActivo ? theme.success : theme.textSecondary,
                            fontSize: 11,
                            fontWeight: FontWeight.w600,
                          ),
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 12),
                  Divider(color: theme.border, height: 1),
                  const SizedBox(height: 12),
                  Row(
                    children: [
                      Expanded(
                        child: _buildMobileInfoItem(
                          'Esquema',
                          proveedor.esquemaActivo == EsquemaPago.push
                              ? 'PUSH NC-Pago'
                              : 'PULL NC-Pago',
                          theme,
                        ),
                      ),
                      Expanded(
                        child: _buildMobileInfoItem(
                          'Días Pago',
                          '${proveedor.diasPago} días',
                          theme,
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 8),
                  Row(
                    children: [
                      Expanded(
                        child: _buildMobileInfoItem(
                          'DPP',
                          proveedor.dppPermitido
                              ? '${proveedor.porcentajeDPPBase.toStringAsFixed(2)} %'
                              : 'No permitido',
                          theme,
                        ),
                      ),
                      Expanded(
                        child: _buildMobileInfoItem(
                          'Contacto',
                          proveedor.contacto,
                          theme,
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 12),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.end,
                    children: [
                      TextButton.icon(
                        onPressed: () => _showProveedorForm(context, proveedor),
                        icon: Icon(Icons.edit, size: 16, color: theme.primary),
                        label: Text(
                          'Editar',
                          style: TextStyle(color: theme.primary),
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }

  Widget _buildMobileInfoItem(String label, String value, AppTheme theme) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          label,
          style: TextStyle(
            color: theme.textSecondary,
            fontSize: 11,
            fontWeight: FontWeight.w500,
          ),
        ),
        const SizedBox(height: 2),
        Text(
          value,
          style: TextStyle(
            color: theme.textPrimary,
            fontSize: 13,
            fontWeight: FontWeight.w600,
          ),
        ),
      ],
    );
  }

  void _showProveedorForm(BuildContext context, Proveedor? proveedor) {
    showDialog(
      context: context,
      builder: (context) => ProveedorFormDialog(proveedor: proveedor),
    );
  }

  void _showProveedorDetails(BuildContext context, Proveedor proveedor) {
    final theme = Theme.of(context).extension<AppTheme>()!;

    showDialog(
      context: context,
      builder: (context) => Dialog(
        child: Container(
          constraints: const BoxConstraints(maxWidth: 500),
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Row(
                children: [
                  Container(
                    width: 60,
                    height: 60,
                    decoration: BoxDecoration(
                      color: theme.primaryBackground,
                      borderRadius: BorderRadius.circular(12),
                      border: Border.all(
                        color: theme.border.withOpacity(0.3),
                        width: 1,
                      ),
                    ),
                    child: ClipRRect(
                      borderRadius: BorderRadius.circular(8),
                      child: proveedor.logoBytes != null
                          ? Image.memory(
                              proveedor.logoBytes!,
                              fit: BoxFit.cover,
                            )
                          : Image.asset(
                              getProveedorLogoPath(proveedor.nombre),
                              fit: BoxFit.cover,
                              errorBuilder: (context, error, stackTrace) {
                                return Center(
                                  child: Text(
                                    proveedor.nombre
                                        .substring(0, 1)
                                        .toUpperCase(),
                                    style: TextStyle(
                                      color: theme.primary,
                                      fontSize: 24,
                                      fontWeight: FontWeight.w600,
                                    ),
                                  ),
                                );
                              },
                            ),
                    ),
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          proveedor.nombre,
                          style: TextStyle(
                            color: theme.textPrimary,
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Text(
                          proveedor.rfc,
                          style: TextStyle(
                            color: theme.textSecondary,
                            fontSize: 14,
                          ),
                        ),
                      ],
                    ),
                  ),
                  IconButton(
                    onPressed: () => Navigator.of(context).pop(),
                    icon: const Icon(Icons.close),
                  ),
                ],
              ),
              const SizedBox(height: 24),
              Divider(color: theme.border),
              const SizedBox(height: 16),
              _buildDetailRow(
                  'Esquema Activo',
                  proveedor.esquemaActivo == EsquemaPago.push
                      ? 'PUSH NC-Pago'
                      : 'PULL NC-Pago',
                  theme),
              _buildDetailRow(
                  'Días de Pago', '${proveedor.diasPago} días', theme),
              _buildDetailRow(
                  'DPP Permitido', proveedor.dppPermitido ? 'Sí' : 'No', theme),
              if (proveedor.dppPermitido) ...[
                _buildDetailRow(
                    '% DPP Base',
                    '${proveedor.porcentajeDPPBase.toStringAsFixed(2)} %',
                    theme),
                _buildDetailRow('Días Gracia DPP',
                    '${proveedor.diasGraciaDPP} días', theme),
              ],
              _buildDetailRow('Contacto', proveedor.contacto, theme),
              _buildDetailRow('Email', proveedor.email, theme),
              _buildDetailRow(
                  'Estado',
                  proveedor.estado == EstadoProveedor.activo
                      ? 'Activo'
                      : 'Inactivo',
                  theme),
              const SizedBox(height: 24),
              Row(
                mainAxisAlignment: MainAxisAlignment.end,
                children: [
                  TextButton(
                    onPressed: () => Navigator.of(context).pop(),
                    child: Text('Cerrar',
                        style: TextStyle(color: theme.textSecondary)),
                  ),
                  const SizedBox(width: 8),
                  ElevatedButton.icon(
                    onPressed: () {
                      Navigator.of(context).pop();
                      _showProveedorForm(context, proveedor);
                    },
                    icon: const Icon(Icons.edit, size: 18),
                    label: const Text('Editar'),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: theme.primary,
                      foregroundColor: Colors.white,
                    ),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildDetailRow(String label, String value, AppTheme theme) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 12),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            width: 140,
            child: Text(
              label,
              style: TextStyle(
                color: theme.textSecondary,
                fontSize: 14,
                fontWeight: FontWeight.w500,
              ),
            ),
          ),
          Expanded(
            child: Text(
              value,
              style: TextStyle(
                color: theme.textPrimary,
                fontSize: 14,
                fontWeight: FontWeight.w600,
              ),
            ),
          ),
        ],
      ),
    );
  }
}
