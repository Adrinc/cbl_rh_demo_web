import 'package:flutter/material.dart';
import 'package:pluto_grid/pluto_grid.dart';

import 'package:facturacion_demo/models/models.dart';
import 'package:facturacion_demo/theme/theme.dart';
import 'package:facturacion_demo/helpers/constants.dart';
import 'package:facturacion_demo/functions/proveedor_logo.dart';

/// ============================================================================
/// PROVEEDOR PLUTO GRID
/// ============================================================================
/// Tabla de proveedores con PlutoGrid para desktop
/// 8 columnas: Proveedor, RFC, Esquema, Días Pago, DPP, % DPP, Contacto, Estado, Acciones
/// ============================================================================
class ProveedorPlutoGrid extends StatefulWidget {
  final List<Proveedor> proveedores;
  final Function(Proveedor) onEdit;
  final Function(Proveedor) onView;

  const ProveedorPlutoGrid({
    super.key,
    required this.proveedores,
    required this.onEdit,
    required this.onView,
  });

  @override
  State<ProveedorPlutoGrid> createState() => _ProveedorPlutoGridState();
}

class _ProveedorPlutoGridState extends State<ProveedorPlutoGrid> {
  late PlutoGridStateManager stateManager;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context).extension<AppTheme>()!;
    final themeMode = Theme.of(context).brightness;

    return Container(
      decoration: BoxDecoration(
        color: theme.surface,
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: theme.border.withOpacity(0.5), width: 1),
        boxShadow: [
          BoxShadow(
            color: theme.textPrimary.withOpacity(0.06),
            blurRadius: 12,
            offset: const Offset(0, 3),
          ),
        ],
      ),
      child: ClipRRect(
        borderRadius: BorderRadius.circular(12),
        child: PlutoGrid(
          key: ValueKey(themeMode),
          columns: _buildColumns(theme),
          rows: [],
          onLoaded: (PlutoGridOnLoadedEvent event) {
            stateManager = event.stateManager;
            _loadData();
          },
          configuration: PlutoGridConfiguration(
            style: PlutoGridStyleConfig(
              gridBackgroundColor: theme.surface,
              rowColor: theme.surface,
              oddRowColor: theme.primaryBackground.withOpacity(0.5),
              activatedColor: theme.primary.withOpacity(0.08),
              gridBorderColor: theme.border,
              borderColor: theme.border,
              activatedBorderColor: theme.primary,
              inactivatedBorderColor: theme.border,
              iconColor: theme.textPrimary,
              disabledIconColor: theme.textSecondary,
              menuBackgroundColor: theme.surface,
              cellTextStyle: TextStyle(color: theme.textPrimary, fontSize: 14),
              columnTextStyle: TextStyle(
                color: theme.textPrimary,
                fontSize: 14,
                fontWeight: FontWeight.w600,
              ),
              cellColorInEditState: theme.primaryBackground,
              cellColorInReadOnlyState: theme.surface,
              columnHeight: 50,
              rowHeight: 75,
              defaultColumnTitlePadding: const EdgeInsets.all(16),
              defaultCellPadding: const EdgeInsets.all(12),
            ),
            columnSize: const PlutoGridColumnSizeConfig(
              autoSizeMode: PlutoAutoSizeMode.scale,
              resizeMode: PlutoResizeMode.normal,
            ),
            localeText: const PlutoGridLocaleText.spanish(),
          ),
          createFooter: (stateManager) {
            return PlutoLazyPagination(
              stateManager: stateManager,
              pageSizeToMove: 15,
              fetch: (request) async => _fetchPage(request.page),
              initialFetch: false,
            );
          },
        ),
      ),
    );
  }

  List<PlutoColumn> _buildColumns(AppTheme theme) {
    return [
      // Proveedor (Nombre con avatar)
      PlutoColumn(
        title: 'Proveedor',
        field: 'proveedor',
        type: PlutoColumnType.text(),
        width: 220,
        renderer: (ctx) {
          final proveedor = ctx.row.cells['_objeto']!.value as Proveedor;
          return Container(
            padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
            child: Row(
              children: [
                // Logo del proveedor (circular)
                CircleAvatar(
                  radius: 18,
                  backgroundColor: theme.primary.withOpacity(0.15),
                  child: ClipOval(
                    child: proveedor.logoBytes != null
                        ? Image.memory(
                            proveedor.logoBytes!,
                            width: 36,
                            height: 36,
                            fit: BoxFit.cover,
                          )
                        : Image.asset(
                            getProveedorLogoPath(proveedor.nombre),
                            width: 36,
                            height: 36,
                            fit: BoxFit.cover,
                            errorBuilder: (context, error, stackTrace) {
                              return Center(
                                child: Text(
                                  proveedor.nombre
                                      .substring(0, 1)
                                      .toUpperCase(),
                                  style: TextStyle(
                                    color: theme.primary,
                                    fontWeight: FontWeight.w600,
                                    fontSize: 14,
                                  ),
                                ),
                              );
                            },
                          ),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: Text(
                    proveedor.nombre,
                    style: TextStyle(
                      color: theme.textPrimary,
                      fontSize: 14,
                      fontWeight: FontWeight.w600,
                    ),
                    overflow: TextOverflow.ellipsis,
                  ),
                ),
              ],
            ),
          );
        },
      ),

      // RFC
      PlutoColumn(
        title: 'RFC',
        field: 'rfc',
        type: PlutoColumnType.text(),
        width: 140,
      ),

      // Esquema
      PlutoColumn(
        title: 'Esquema',
        field: 'esquema',
        type: PlutoColumnType.text(),
        width: 150,
        renderer: (ctx) {
          final esquema = ctx.cell.value as String;
          final isPush = esquema == EsquemaPago.push;
          return Container(
            alignment: Alignment.center,
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
              decoration: BoxDecoration(
                color: isPush
                    ? theme.primary.withOpacity(0.15)
                    : theme.secondary.withOpacity(0.15),
                borderRadius: BorderRadius.circular(20),
                border: Border.all(
                  color: isPush ? theme.primary : theme.secondary,
                  width: 1,
                ),
              ),
              child: Text(
                isPush ? 'PUSH NC-Pago' : 'PULL NC-Pago',
                style: TextStyle(
                  color: isPush ? theme.primary : theme.secondary,
                  fontWeight: FontWeight.w600,
                  fontSize: 11,
                ),
              ),
            ),
          );
        },
      ),

      // Días Pago
      PlutoColumn(
        title: 'Días Pago',
        field: 'diasPago',
        type: PlutoColumnType.number(),
        width: 110,
        renderer: (ctx) {
          final dias = ctx.cell.value as int;
          return Container(
            alignment: Alignment.center,
            child: Text(
              '$dias días',
              style: TextStyle(
                color: theme.textPrimary,
                fontSize: 14,
                fontWeight: FontWeight.w600,
              ),
            ),
          );
        },
      ),

      // DPP Permitido
      PlutoColumn(
        title: 'DPP',
        field: 'dppPermitido',
        type: PlutoColumnType.text(),
        width: 100,
        renderer: (ctx) {
          final permitido = ctx.cell.value as bool;
          return Container(
            alignment: Alignment.center,
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
              decoration: BoxDecoration(
                color: permitido
                    ? theme.success.withOpacity(0.15)
                    : theme.textSecondary.withOpacity(0.15),
                borderRadius: BorderRadius.circular(12),
              ),
              child: Text(
                permitido ? 'SÍ' : 'NO',
                style: TextStyle(
                  color: permitido ? theme.success : theme.textSecondary,
                  fontWeight: FontWeight.w600,
                  fontSize: 11,
                ),
              ),
            ),
          );
        },
      ),

      // % DPP Base
      PlutoColumn(
        title: '% DPP Base',
        field: 'porcentajeDPP',
        type: PlutoColumnType.number(),
        width: 110,
        renderer: (ctx) {
          final proveedor = ctx.row.cells['_objeto']!.value as Proveedor;
          if (!proveedor.dppPermitido) {
            return Container(
              alignment: Alignment.center,
              child: Text(
                '--',
                style: TextStyle(
                  color: theme.textSecondary,
                  fontSize: 14,
                ),
              ),
            );
          }
          final porcentaje = ctx.cell.value as double;
          return Container(
            alignment: Alignment.center,
            child: Text(
              '${porcentaje.toStringAsFixed(2)} %',
              style: TextStyle(
                color: theme.secondary,
                fontSize: 14,
                fontWeight: FontWeight.w600,
              ),
            ),
          );
        },
      ),

      // Contacto
      PlutoColumn(
        title: 'Contacto',
        field: 'contacto',
        type: PlutoColumnType.text(),
        width: 150,
      ),

      // Estado
      PlutoColumn(
        title: 'Estado',
        field: 'estado',
        type: PlutoColumnType.text(),
        width: 110,
        renderer: (ctx) {
          final estadoValue = ctx.cell.value;
          // Manejar tanto String como EstadoProveedor
          final bool isActivo;
          if (estadoValue is EstadoProveedor) {
            isActivo = estadoValue == EstadoProveedor.activo;
          } else if (estadoValue is String) {
            isActivo = estadoValue == 'activo' ||
                estadoValue.toLowerCase() == 'activo';
          } else {
            isActivo = true; // Fallback
          }

          return Container(
            alignment: Alignment.center,
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
              decoration: BoxDecoration(
                color: isActivo
                    ? theme.success.withOpacity(0.15)
                    : theme.textSecondary.withOpacity(0.15),
                borderRadius: BorderRadius.circular(20),
              ),
              child: Text(
                isActivo ? 'ACTIVO' : 'INACTIVO',
                style: TextStyle(
                  color: isActivo ? theme.success : theme.textSecondary,
                  fontWeight: FontWeight.w600,
                  fontSize: 11,
                ),
              ),
            ),
          );
        },
      ),

      // Acciones
      PlutoColumn(
        title: 'Acciones',
        field: 'acciones',
        type: PlutoColumnType.text(),
        width: 140,
        renderer: (ctx) {
          final proveedor = ctx.row.cells['_objeto']!.value as Proveedor;
          return Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              IconButton(
                onPressed: () => widget.onView(proveedor),
                icon: Icon(Icons.visibility_outlined,
                    color: theme.primary, size: 18),
                tooltip: 'Ver detalles',
              ),
              const SizedBox(width: 4),
              IconButton(
                onPressed: () => widget.onEdit(proveedor),
                icon:
                    Icon(Icons.edit_outlined, color: theme.secondary, size: 18),
                tooltip: 'Editar',
              ),
            ],
          );
        },
      ),

      // Columna oculta para objeto completo
      PlutoColumn(
        title: '_objeto',
        field: '_objeto',
        type: PlutoColumnType.text(),
        hide: true,
      ),
    ];
  }

  void _loadData() {
    stateManager.removeAllRows();
    final rows = widget.proveedores.map((proveedor) {
      return PlutoRow(
        cells: {
          'proveedor': PlutoCell(value: proveedor.nombre),
          'rfc': PlutoCell(value: proveedor.rfc),
          'esquema': PlutoCell(value: proveedor.esquemaActivo),
          'diasPago': PlutoCell(value: proveedor.diasPago),
          'dppPermitido': PlutoCell(value: proveedor.dppPermitido),
          'porcentajeDPP': PlutoCell(value: proveedor.porcentajeDPPBase),
          'contacto': PlutoCell(value: proveedor.contacto),
          'estado': PlutoCell(value: proveedor.estado),
          'acciones': PlutoCell(value: ''),
          '_objeto': PlutoCell(value: proveedor),
        },
      );
    }).toList();

    stateManager.appendRows(rows);
  }

  Future<PlutoLazyPaginationResponse> _fetchPage(int page) async {
    final startIndex = page * 15;
    final endIndex = (startIndex + 15).clamp(0, widget.proveedores.length);

    if (startIndex >= widget.proveedores.length) {
      return PlutoLazyPaginationResponse(
        totalPage: (widget.proveedores.length / 15).ceil(),
        rows: [],
      );
    }

    final pageProveedores = widget.proveedores.sublist(startIndex, endIndex);
    final rows = pageProveedores.map((proveedor) {
      return PlutoRow(
        cells: {
          'proveedor': PlutoCell(value: proveedor.nombre),
          'rfc': PlutoCell(value: proveedor.rfc),
          'esquema': PlutoCell(value: proveedor.esquemaActivo),
          'diasPago': PlutoCell(value: proveedor.diasPago),
          'dppPermitido': PlutoCell(value: proveedor.dppPermitido),
          'porcentajeDPP': PlutoCell(value: proveedor.porcentajeDPPBase),
          'contacto': PlutoCell(value: proveedor.contacto),
          'estado': PlutoCell(value: proveedor.estado),
          'acciones': PlutoCell(value: ''),
          '_objeto': PlutoCell(value: proveedor),
        },
      );
    }).toList();

    return PlutoLazyPaginationResponse(
      totalPage: (widget.proveedores.length / 15).ceil(),
      rows: rows,
    );
  }
}
