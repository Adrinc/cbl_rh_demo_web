import 'package:flutter/material.dart';
import 'package:pluto_grid/pluto_grid.dart';
import 'package:nethive_neo/providers/nethive/componentes_provider.dart';
import 'package:nethive_neo/theme/theme.dart';
import 'inventario_empty_state.dart';

class InventarioComponentsTable extends StatelessWidget {
  final ComponentesProvider componentesProvider;
  final Function(dynamic componente, ComponentesProvider provider)
      onShowDetails;
  final Function(dynamic componente, ComponentesProvider provider)
      onEditComponent;
  final Function(dynamic componente, ComponentesProvider provider)
      onDeleteComponent;
  final Function(String categoryName) getCategoryIcon;

  const InventarioComponentsTable({
    super.key,
    required this.componentesProvider,
    required this.onShowDetails,
    required this.onEditComponent,
    required this.onDeleteComponent,
    required this.getCategoryIcon,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        color: AppTheme.of(context).secondaryBackground,
        borderRadius: BorderRadius.circular(16),
        border: Border.all(
          color: AppTheme.of(context).primaryColor.withOpacity(0.2),
        ),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.05),
            blurRadius: 10,
            offset: const Offset(0, 4),
          ),
        ],
      ),
      child: Column(
        children: [
          // Header de la tabla
          Container(
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              gradient: AppTheme.of(context).modernGradient,
              borderRadius: const BorderRadius.only(
                topLeft: Radius.circular(16),
                topRight: Radius.circular(16),
              ),
            ),
            child: Row(
              children: [
                Container(
                  padding: const EdgeInsets.all(8),
                  decoration: BoxDecoration(
                    color: Colors.white.withOpacity(0.2),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: const Icon(
                    Icons.table_view,
                    color: Colors.white,
                    size: 20,
                  ),
                ),
                const SizedBox(width: 16),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'Tabla de Componentes',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 16,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      Text(
                        'Visualización detallada de todos los componentes',
                        style: TextStyle(
                          color: Colors.white.withOpacity(0.8),
                          fontSize: 12,
                        ),
                      ),
                    ],
                  ),
                ),
                Container(
                  padding:
                      const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
                  decoration: BoxDecoration(
                    color: Colors.white.withOpacity(0.2),
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Text(
                    '${componentesProvider.componentes.length} elementos',
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 12,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ],
            ),
          ),

          // Tabla de componentes con PlutoGrid
          Expanded(
            child: componentesProvider.componentesRows.isEmpty
                ? const InventarioEmptyState()
                : PlutoGrid(
                    key: UniqueKey(),
                    configuration: PlutoGridConfiguration(
                      enableMoveDownAfterSelecting: true,
                      enableMoveHorizontalInEditing: true,
                      localeText: const PlutoGridLocaleText.spanish(),
                      scrollbar: PlutoGridScrollbarConfig(
                        draggableScrollbar: true,
                        isAlwaysShown: false,
                        onlyDraggingThumb: true,
                        enableScrollAfterDragEnd: true,
                        scrollbarThickness: 12,
                        scrollbarThicknessWhileDragging: 16,
                        hoverWidth: 20,
                        scrollBarColor:
                            AppTheme.of(context).primaryColor.withOpacity(0.7),
                        scrollBarTrackColor: Colors.grey.withOpacity(0.2),
                        scrollbarRadius: const Radius.circular(8),
                        scrollbarRadiusWhileDragging: const Radius.circular(10),
                      ),
                      style: PlutoGridStyleConfig(
                        enableRowColorAnimation: true,
                        gridBorderColor:
                            AppTheme.of(context).primaryColor.withOpacity(0.5),
                        disabledIconColor:
                            AppTheme.of(context).alternate.withOpacity(0.3),
                        iconColor:
                            AppTheme.of(context).alternate.withOpacity(0.3),
                        activatedBorderColor: AppTheme.of(context).primaryColor,
                        inactivatedBorderColor: Colors.grey.withOpacity(0.3),
                        gridBackgroundColor:
                            AppTheme.of(context).primaryBackground,
                        rowColor: AppTheme.of(context).secondaryBackground,
                        activatedColor:
                            AppTheme.of(context).primaryColor.withOpacity(0.1),
                        checkedColor:
                            AppTheme.of(context).primaryColor.withOpacity(0.2),
                        cellTextStyle: TextStyle(
                          color: AppTheme.of(context).primaryText,
                          fontSize: 13,
                        ),
                        columnTextStyle: const TextStyle(
                          color: Colors.white,
                          fontSize: 13,
                          fontWeight: FontWeight.bold,
                        ),
                        menuBackgroundColor:
                            AppTheme.of(context).secondaryBackground,
                        gridBorderRadius: const BorderRadius.only(
                          bottomLeft: Radius.circular(16),
                          bottomRight: Radius.circular(16),
                        ),
                        rowHeight: 70,
                      ),
                      columnFilter: const PlutoGridColumnFilterConfig(
                        filters: [
                          ...FilterHelper.defaultFilters,
                        ],
                      ),
                    ),
                    columns: _buildColumns(context),
                    rows: componentesProvider.componentesRows,
                    onLoaded: (event) {
                      componentesProvider.componentesStateManager =
                          event.stateManager;
                    },
                    createFooter: (stateManager) {
                      return PlutoPagination(stateManager);
                    },
                  ),
          ),
        ],
      ),
    );
  }

  List<PlutoColumn> _buildColumns(BuildContext context) {
    return [
      PlutoColumn(
        title: 'Nu.',
        field: 'numero_fila',
        width: 60,
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.center,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          return Container(
            padding: const EdgeInsets.all(8),
            child: Center(
              child: Text(
                (rendererContext.rowIdx + 1).toString(),
                style: TextStyle(
                  color: AppTheme.of(context).primaryText,
                  fontSize: 13,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'RFID',
        field: 'rfid',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.center,
        width: 150,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          final rfid = rendererContext.cell.value?.toString();
          return Container(
            padding: const EdgeInsets.all(8),
            child: Center(
              child: rfid != null && rfid.isNotEmpty
                  ? Container(
                      padding: const EdgeInsets.symmetric(
                          horizontal: 8, vertical: 4),
                      decoration: BoxDecoration(
                        color: Colors.indigo.withOpacity(0.1),
                        borderRadius: BorderRadius.circular(8),
                        border: Border.all(
                          color: Colors.indigo.withOpacity(0.3),
                        ),
                      ),
                      child: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          const Icon(
                            Icons.nfc,
                            color: Colors.indigo,
                            size: 12,
                          ),
                          const SizedBox(width: 4),
                          Text(
                            rfid,
                            style: const TextStyle(
                              color: Colors.indigo,
                              fontSize: 11,
                              fontWeight: FontWeight.w600,
                              fontFamily: 'monospace',
                            ),
                          ),
                        ],
                      ),
                    )
                  : Text(
                      'Sin RFID',
                      style: TextStyle(
                        color: AppTheme.of(context).secondaryText,
                        fontSize: 11,
                        fontStyle: FontStyle.italic,
                      ),
                    ),
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'Componente',
        field: 'nombre',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.left,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          return Container(
            padding: const EdgeInsets.all(8),
            child: Row(
              children: [
                // Imagen del componente
                Container(
                  width: 32,
                  height: 32,
                  decoration: BoxDecoration(
                    color: AppTheme.of(context).primaryColor.withOpacity(0.1),
                    borderRadius: BorderRadius.circular(6),
                  ),
                  child:
                      rendererContext.row.cells['imagen_url']?.value != null &&
                              rendererContext.row.cells['imagen_url']!.value
                                  .toString()
                                  .isNotEmpty
                          ? ClipRRect(
                              borderRadius: BorderRadius.circular(6),
                              child: Image.network(
                                rendererContext.row.cells['imagen_url']!.value
                                    .toString(),
                                fit: BoxFit.cover,
                                errorBuilder: (context, error, stackTrace) {
                                  return Icon(
                                    Icons.devices,
                                    color: AppTheme.of(context).primaryColor,
                                    size: 16,
                                  );
                                },
                              ),
                            )
                          : Icon(
                              Icons.devices,
                              color: AppTheme.of(context).primaryColor,
                              size: 16,
                            ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: Text(
                    rendererContext.cell.value.toString(),
                    style: TextStyle(
                      color: AppTheme.of(context).primaryText,
                      fontSize: 13,
                      fontWeight: FontWeight.w600,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                ),
              ],
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'Categoría',
        field: 'categoria_nombre',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.center,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          final categoriaNombre = rendererContext.cell.value.toString();
          final colorCategoria =
              rendererContext.row.cells['color_categoria']?.value?.toString();

          // Convertir color hexadecimal a Color
          Color categoriaColor = AppTheme.of(context).primaryColor;
          if (colorCategoria != null && colorCategoria.isNotEmpty) {
            try {
              final hexColor = colorCategoria.replaceAll('#', '');
              categoriaColor = Color(int.parse('FF$hexColor', radix: 16));
            } catch (e) {
              // Si hay error al parsear el color, usar color por defecto
              categoriaColor = AppTheme.of(context).primaryColor;
            }
          }

          // Obtener icono según la categoría
          IconData categoriaIcon = getCategoryIcon(categoriaNombre);

          return Container(
            padding: const EdgeInsets.all(8),
            child: Center(
              child: Container(
                padding:
                    const EdgeInsets.symmetric(horizontal: 10, vertical: 6),
                decoration: BoxDecoration(
                  color: categoriaColor.withOpacity(0.15),
                  borderRadius: BorderRadius.circular(10),
                  border: Border.all(
                    color: categoriaColor.withOpacity(0.4),
                    width: 1.5,
                  ),
                ),
                child: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Icon(
                      categoriaIcon,
                      color: categoriaColor,
                      size: 14,
                    ),
                    const SizedBox(width: 6),
                    Text(
                      categoriaNombre,
                      style: TextStyle(
                        color: categoriaColor,
                        fontSize: 11,
                        fontWeight: FontWeight.bold,
                      ),
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'Estado',
        field: 'activo',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.center,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          final isActivo = rendererContext.cell.value.toString() == 'Sí';
          return Container(
            padding: const EdgeInsets.all(8),
            child: Center(
              child: Container(
                padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                decoration: BoxDecoration(
                  color:
                      (isActivo ? Colors.green : Colors.red).withOpacity(0.1),
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Icon(
                      isActivo ? Icons.check_circle : Icons.cancel,
                      color: isActivo ? Colors.green : Colors.red,
                      size: 12,
                    ),
                    const SizedBox(width: 4),
                    Text(
                      isActivo ? 'Activo' : 'Inactivo',
                      style: TextStyle(
                        color: isActivo ? Colors.green : Colors.red,
                        fontSize: 11,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'En Uso',
        field: 'en_uso',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.center,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          final enUso = rendererContext.cell.value.toString() == 'Sí';
          return Container(
            padding: const EdgeInsets.all(8),
            child: Center(
              child: Container(
                padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                decoration: BoxDecoration(
                  color: (enUso ? Colors.orange : Colors.grey).withOpacity(0.1),
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Text(
                  enUso ? 'En Uso' : 'Libre',
                  style: TextStyle(
                    color: enUso ? Colors.orange : Colors.grey,
                    fontSize: 11,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'Ubicación',
        field: 'ubicacion',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.left,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          return Container(
            padding: const EdgeInsets.all(8),
            child: Row(
              children: [
                Icon(
                  Icons.location_on,
                  color: AppTheme.of(context).primaryColor,
                  size: 14,
                ),
                const SizedBox(width: 4),
                Expanded(
                  child: Text(
                    rendererContext.cell.value.toString().isEmpty
                        ? 'Sin ubicación'
                        : rendererContext.cell.value.toString(),
                    style: TextStyle(
                      color: AppTheme.of(context).primaryText,
                      fontSize: 12,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                ),
              ],
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'Descripción',
        field: 'descripcion',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.left,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          return Container(
            padding: const EdgeInsets.all(8),
            child: Text(
              rendererContext.cell.value.toString().isEmpty
                  ? 'Sin descripción'
                  : rendererContext.cell.value.toString(),
              style: TextStyle(
                color: AppTheme.of(context).secondaryText,
                fontSize: 12,
              ),
              maxLines: 2,
              overflow: TextOverflow.ellipsis,
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'Fecha Registro',
        field: 'fecha_registro',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.center,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          return Container(
            padding: const EdgeInsets.all(8),
            child: Text(
              rendererContext.cell.value.toString(),
              textAlign: TextAlign.center,
              style: TextStyle(
                color: AppTheme.of(context).secondaryText,
                fontSize: 11,
              ),
            ),
          );
        },
      ),
      PlutoColumn(
        title: 'Acciones',
        field: 'editar',
        titleTextAlign: PlutoColumnTextAlign.center,
        textAlign: PlutoColumnTextAlign.center,
        type: PlutoColumnType.text(),
        enableEditingMode: false,
        backgroundColor: AppTheme.of(context).primaryColor,
        enableContextMenu: false,
        enableDropToResize: false,
        renderer: (rendererContext) {
          final componenteId =
              rendererContext.row.cells['id']?.value.toString() ?? '';
          final componente =
              componentesProvider.getComponenteById(componenteId);

          return Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Botón ver detalles
              Tooltip(
                message: 'Ver detalles',
                child: InkWell(
                  onTap: () => onShowDetails(componente, componentesProvider),
                  child: Container(
                    padding: const EdgeInsets.all(4),
                    decoration: BoxDecoration(
                      color: Colors.blue,
                      borderRadius: BorderRadius.circular(4),
                    ),
                    child: const Icon(
                      Icons.visibility,
                      color: Colors.white,
                      size: 14,
                    ),
                  ),
                ),
              ),
              const SizedBox(width: 4),
              // Botón editar
              Tooltip(
                message: 'Editar',
                child: InkWell(
                  onTap: () => onEditComponent(componente, componentesProvider),
                  child: Container(
                    padding: const EdgeInsets.all(4),
                    decoration: BoxDecoration(
                      color: AppTheme.of(context).primaryColor,
                      borderRadius: BorderRadius.circular(4),
                    ),
                    child: const Icon(
                      Icons.edit,
                      color: Colors.white,
                      size: 14,
                    ),
                  ),
                ),
              ),
              const SizedBox(width: 4),
              // Botón eliminar
              Tooltip(
                message: 'Eliminar',
                child: InkWell(
                  onTap: () =>
                      onDeleteComponent(componente, componentesProvider),
                  child: Container(
                    padding: const EdgeInsets.all(4),
                    decoration: BoxDecoration(
                      color: Colors.red,
                      borderRadius: BorderRadius.circular(4),
                    ),
                    child: const Icon(
                      Icons.delete,
                      color: Colors.white,
                      size: 14,
                    ),
                  ),
                ),
              ),
            ],
          );
        },
      ),
    ];
  }
}
