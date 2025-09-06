# ClaseV

import sys
import os
import re
import csv
from collections import Counter

from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout, QGridLayout,
    QLabel, QLineEdit, QPushButton, QTextEdit, QComboBox, QMessageBox,
    QFileDialog, QGroupBox, QListWidget, QListWidgetItem, QSplitter
)
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QFont


class SistemaDocentes(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Sistema de Gestión de Docentes")
        self.setGeometry(100, 100, 1000, 700)

        # Archivo donde se guardarán los datos
        self.archivo_datos = "docentes.txt"

        # Estado de edición (None si estamos agregando)
        self.item_en_edicion = None

        # Configurar interfaz
        self.configurar_interfaz()
        self.cargar_datos()
        
        self.setStyleSheet("""
 
    QMainWindow { 
        border-image: url('c:/Users/florp/OneDrive/Escritorio/interfaz/app.png') 0 0 0 0 stretch stretch; 
    }

    QGroupBox {
        font-weight: bold; 
        border: 2px solid #ffffff;
        border-radius: 8px; 
        margin: 10px; 
        padding-top: 25px;
        font-size: 16px;
        color: #ffffff;
    }
    QGroupBox::title {
        subcontrol-origin: margin; 
        left: 15px;
        padding: 4px 12px;
        background-color: rgba(0,0,0,0.7);
        border: 1px solid #ffffff;
        border-radius: 6px;
        font-size: 14px;
        font-weight: bold;
        color: #ffffff;
    }

    QPushButton {
        background-color: #222222;
        color: #ffffff;
        border: 2px solid #ffffff;
        padding: 10px 18px;
        border-radius: 6px;
        font-weight: bold;
        font-size: 14px;
    }
    QPushButton:hover { background-color: #444444; }
    QPushButton:pressed { background-color: #666666; }

    QLineEdit, QComboBox {
        padding: 8px;
        border: 1px solid #ffffff;
        border-radius: 6px;
        background-color: #111111;
        font-size: 14px;
        color: #ffffff;
    }
    QLineEdit:focus, QComboBox:focus { border: 2px solid #ffffff; }

    QListWidget, QTextEdit {
        border: 1px solid #ffffff;
        border-radius: 6px;
        background-color: #111111;
        font-size: 14px;
        color: #ffffff;
    }
    
    QLabel {
    font-size: 16px;    
    font-weight: bold;    
    color: #000000;
    }
""")


    #Interfaz
    def configurar_interfaz(self):
        central_widget = QWidget()
        self.setCentralWidget(central_widget)

        main_layout = QHBoxLayout()
        central_widget.setLayout(main_layout)

        splitter = QSplitter(Qt.Horizontal)
        main_layout.addWidget(splitter)

        panel_izquierdo = self.crear_panel_formulario()
        splitter.addWidget(panel_izquierdo)

        panel_derecho = self.crear_panel_lista()
        splitter.addWidget(panel_derecho)

        splitter.setSizes([420, 580])

    def crear_panel_formulario(self):
        widget = QWidget()
        layout = QVBoxLayout()

        #Formulario
        grupo_form = QGroupBox("Datos del Docente")
        form_layout = QGridLayout()

        self.legajo_edit = QLineEdit()
        self.legajo_edit.setPlaceholderText("Ej: DOC001")
        form_layout.addWidget(QLabel("Legajo:"), 0, 0)
        form_layout.addWidget(self.legajo_edit, 0, 1)

        self.nombre_edit = QLineEdit()
        form_layout.addWidget(QLabel("Nombre:"), 1, 0)
        form_layout.addWidget(self.nombre_edit, 1, 1)

        self.apellido_edit = QLineEdit()
        form_layout.addWidget(QLabel("Apellido:"), 2, 0)
        form_layout.addWidget(self.apellido_edit, 2, 1)

        self.dni_edit = QLineEdit()
        form_layout.addWidget(QLabel("DNI:"), 3, 0)
        form_layout.addWidget(self.dni_edit, 3, 1)

        self.email_edit = QLineEdit()
        form_layout.addWidget(QLabel("Email:"), 4, 0)
        form_layout.addWidget(self.email_edit, 4, 1)

        self.telefono_edit = QLineEdit()
        form_layout.addWidget(QLabel("Teléfono:"), 5, 0)
        form_layout.addWidget(self.telefono_edit, 5, 1)

        self.materia_edit = QLineEdit()
        form_layout.addWidget(QLabel("Materia:"), 6, 0)
        form_layout.addWidget(self.materia_edit, 6, 1)

        self.categoria_combo = QComboBox()
        self.categoria_combo.addItems(["Titular", "Asociado", "Adjunto", "Auxiliar", "Interino"])
        form_layout.addWidget(QLabel("Categoría:"), 7, 0)
        form_layout.addWidget(self.categoria_combo, 7, 1)

        grupo_form.setLayout(form_layout)
        layout.addWidget(grupo_form)

        #Botones
        grupo_botones = QGroupBox("Acciones")
        botones_layout = QVBoxLayout()

        self.btn_agregar = QPushButton("Agregar Docente")
        self.btn_agregar.clicked.connect(self.agregar_docente)
        botones_layout.addWidget(self.btn_agregar)

        self.btn_buscar = QPushButton("Buscar por Legajo")
        self.btn_buscar.clicked.connect(self.buscar_docente)
        botones_layout.addWidget(self.btn_buscar)

        self.btn_modificar = QPushButton("Modificar Seleccionado")
        self.btn_modificar.clicked.connect(self.modificar_docente)
        botones_layout.addWidget(self.btn_modificar)

        self.btn_eliminar = QPushButton("Eliminar Seleccionado")
        self.btn_eliminar.clicked.connect(self.eliminar_docente)
        botones_layout.addWidget(self.btn_eliminar)

        self.btn_limpiar = QPushButton("Limpiar Formulario")
        self.btn_limpiar.clicked.connect(self.limpiar_formulario)
        botones_layout.addWidget(self.btn_limpiar)

        self.btn_exportar = QPushButton("Exportar CSV…")
        self.btn_exportar.clicked.connect(self.exportar_datos)
        botones_layout.addWidget(self.btn_exportar)

        self.btn_estadisticas = QPushButton("Estadísticas por Categoría")
        self.btn_estadisticas.clicked.connect(self.mostrar_estadisticas)
        botones_layout.addWidget(self.btn_estadisticas)

        grupo_botones.setLayout(botones_layout)
        layout.addWidget(grupo_botones)

        widget.setLayout(layout)
        return widget

    def crear_panel_lista(self):
        widget = QWidget()
        layout = QVBoxLayout()

        # Búsqueda
        busqueda_layout = QHBoxLayout()
        busqueda_layout.addWidget(QLabel("Buscar:"))
        self.busqueda_edit = QLineEdit()
        self.busqueda_edit.setPlaceholderText("Apellido, nombre o legajo…")
        self.busqueda_edit.textChanged.connect(self.filtrar_lista)
        busqueda_layout.addWidget(self.busqueda_edit)
        layout.addLayout(busqueda_layout)

        # Lista
        self.lista_docentes = QListWidget()
        self.lista_docentes.itemClicked.connect(self.mostrar_detalles)
        layout.addWidget(self.lista_docentes)

        # Detalles
        grupo_detalles = QGroupBox("Detalles del Docente Seleccionado")
        self.detalles_text = QTextEdit()
        self.detalles_text.setReadOnly(True)
        self.detalles_text.setMaximumHeight(220)
        detalles_layout = QVBoxLayout()
        detalles_layout.addWidget(self.detalles_text)
        grupo_detalles.setLayout(detalles_layout)
        layout.addWidget(grupo_detalles)

        widget.setLayout(layout)
        return widget

    
    def cargar_datos(self):
        """Cargar datos desde el archivo TXT."""
        if not os.path.exists(self.archivo_datos):
            return
        try:
            with open(self.archivo_datos, 'r', encoding='utf-8') as archivo:
                for linea in archivo:
                    if not linea.strip():
                        continue
                    datos = linea.rstrip('\n').split('|')
                    if len(datos) == 8:
                        self.agregar_a_lista(datos)
        except Exception as e:
            QMessageBox.critical(self, 'Error', f'Error al cargar datos:\n{e}')

    def guardar_datos(self):
        """Guardar todos los datos al archivo TXT."""
        try:
            with open(self.archivo_datos, 'w', encoding='utf-8') as archivo:
                for i in range(self.lista_docentes.count()):
                    item = self.lista_docentes.item(i)
                    datos = item.data(Qt.UserRole)
                    archivo.write('|'.join(datos) + '\n')
        except Exception as e:
            QMessageBox.critical(self, 'Error', f'Error al guardar datos:\n{e}')

   
    def validar_email(self, email: str) -> bool:
        patron = r'^[\w\.-]+@[\w\.-]+\.\w{2,}$'
        return re.match(patron, email) is not None

    def validar_telefono(self, tel: str) -> bool:
        # Acepta dígitos, espacios, +, -, paréntesis; requiere al menos 6 dígitos
        solo_digitos = re.sub(r'\D', '', tel)
        return len(solo_digitos) >= 6

    def legajo_existe(self, legajo: str, ignorar_item: QListWidgetItem = None) -> bool:
        legajo_l = legajo.strip().lower()
        for i in range(self.lista_docentes.count()):
            item = self.lista_docentes.item(i)
            if ignorar_item is not None and item is ignorar_item:
                continue
            datos = item.data(Qt.UserRole)
            if datos[0].strip().lower() == legajo_l:
                return True
        return False

    def datos_formulario(self):
        return [
            self.legajo_edit.text().strip(),
            self.nombre_edit.text().strip(),
            self.apellido_edit.text().strip(),
            self.dni_edit.text().strip(),
            self.email_edit.text().strip(),
            self.telefono_edit.text().strip(),
            self.materia_edit.text().strip(),
            self.categoria_combo.currentText()
        ]

    def set_formulario(self, datos):
        self.legajo_edit.setText(datos[0])
        self.nombre_edit.setText(datos[1])
        self.apellido_edit.setText(datos[2])
        self.dni_edit.setText(datos[3])
        self.email_edit.setText(datos[4])
        self.telefono_edit.setText(datos[5])
        self.materia_edit.setText(datos[6])
        idx = self.categoria_combo.findText(datos[7])
        self.categoria_combo.setCurrentIndex(idx if idx >= 0 else 0)

   
    def agregar_a_lista(self, datos):
        """Agregar un docente a la lista (datos = [legajo,...,categoria])."""
        texto_item = f"{datos[2]}, {datos[1]} ({datos[0]})"
        item = QListWidgetItem(texto_item)
        item.setData(Qt.UserRole, datos)
        self.lista_docentes.addItem(item)

    def filtrar_lista(self):
        texto_busqueda = self.busqueda_edit.text().lower().strip()
        for i in range(self.lista_docentes.count()):
            item = self.lista_docentes.item(i)
            datos = item.data(Qt.UserRole)
            coincide = (
                texto_busqueda in datos[0].lower() or
                texto_busqueda in datos[1].lower() or
                texto_busqueda in datos[2].lower()
            )
            item.setHidden(not coincide)

    def mostrar_detalles(self, item):
        datos = item.data(Qt.UserRole)
        detalles = (
            "INFORMACIÓN DEL DOCENTE\n"
            "========================\n"
            f"Legajo: {datos[0]}\n"
            f"Nombre: {datos[1]}\n"
            f"Apellido: {datos[2]}\n"
            f"DNI: {datos[3]}\n"
            f"Email: {datos[4]}\n"
            f"Teléfono: {datos[5]}\n"
            f"Materia: {datos[6]}\n"
            f"Categoría: {datos[7]}"
        )
        self.detalles_text.setPlainText(detalles)
        

   
    def agregar_docente(self):
        """Validar y agregar un nuevo docente."""
        datos = self.datos_formulario()
        legajo, nombre, apellido, dni, email, tel, materia, categoria = datos

        # Validaciones
        if not legajo or not nombre or not apellido:
            QMessageBox.warning(self, 'Campos requeridos',
                                'Legajo, Nombre y Apellido son obligatorios.')
        elif self.legajo_existe(legajo):
            QMessageBox.warning(self, 'Duplicado',
                                'Ya existe un docente con ese legajo.')
        elif email and not self.validar_email(email):
            QMessageBox.warning(self, 'Email no válido',
                                'Ingrese un email válido (o deje vacío).')
        elif tel and not self.validar_telefono(tel):
            QMessageBox.warning(self, 'Teléfono no válido',
                                'El teléfono debe contener al menos 6 dígitos.')
        else:
            self.agregar_a_lista(datos)
            self.guardar_datos()
            self.limpiar_formulario()
            QMessageBox.information(self, 'Éxito', 'Docente agregado correctamente.')

    def buscar_docente(self):
        """Buscar por legajo usando el campo Legajo."""
        legajo = self.legajo_edit.text().strip()
        if not legajo:
            QMessageBox.warning(self, 'Error', 'Ingrese un legajo para buscar.')
            return

        for i in range(self.lista_docentes.count()):
            item = self.lista_docentes.item(i)
            datos = item.data(Qt.UserRole)
            if datos[0].strip().lower() == legajo.lower():
                self.lista_docentes.setCurrentItem(item)
                self.mostrar_detalles(item)
                return

        QMessageBox.information(self, 'No encontrado',
                                f'No se encontró docente con legajo: {legajo}')

    def modificar_docente(self):
        """Cargar en el formulario el docente seleccionado y pasar a modo edición."""
        item_actual = self.lista_docentes.currentItem()
        if not item_actual:
            QMessageBox.warning(self, 'Error', 'Seleccione un docente para modificar.')
            return

        self.item_en_edicion = item_actual
        datos = item_actual.data(Qt.UserRole)
        self.set_formulario(datos)

        # Cambiar botón a "Actualizar"
        self.btn_agregar.setText("Actualizar Docente")
        try:
            self.btn_agregar.clicked.disconnect()
        except TypeError:
            pass
        self.btn_agregar.clicked.connect(self.actualizar_docente)

    def actualizar_docente(self):
        """Actualizar datos del docente en edición."""
        if self.item_en_edicion is None:
            return

        nuevos = self.datos_formulario()
        legajo, nombre, apellido, dni, email, tel, materia, categoria = nuevos

        # Validaciones
        if not legajo or not nombre or not apellido:
            QMessageBox.warning(self, 'Campos requeridos',
                                'Legajo, Nombre y Apellido son obligatorios.')
            return
        if self.legajo_existe(legajo, ignorar_item=self.item_en_edicion):
            QMessageBox.warning(self, 'Duplicado',
                                'Ya existe otro docente con ese legajo.')
            return
        if email and not self.validar_email(email):
            QMessageBox.warning(self, 'Email no válido',
                                'Ingrese un email válido (o deje vacío).')
            return
        if tel and not self.validar_telefono(tel):
            QMessageBox.warning(self, 'Teléfono no válido',
                                'El teléfono debe contener al menos 6 dígitos.')
            return

        # Actualizar item
        self.item_en_edicion.setData(Qt.UserRole, nuevos)
        self.item_en_edicion.setText(f"{apellido}, {nombre} ({legajo})")
        self.guardar_datos()
        self.mostrar_detalles(self.item_en_edicion)

        # Restaurar estado
        self.item_en_edicion = None
        self.btn_agregar.setText("Agregar Docente")
        try:
            self.btn_agregar.clicked.disconnect()
        except TypeError:
            pass
        self.btn_agregar.clicked.connect(self.agregar_docente)
        QMessageBox.information(self, 'Éxito', 'Docente actualizado correctamente.')
        self.limpiar_formulario()

    def eliminar_docente(self):
        item_actual = self.lista_docentes.currentItem()
        if not item_actual:
            QMessageBox.warning(self, 'Error', 'Seleccione un docente para eliminar.')
            return

        datos = item_actual.data(Qt.UserRole)
        respuesta = QMessageBox.question(
            self, 'Confirmar eliminación',
            f'¿Está seguro de eliminar a {datos[1]} {datos[2]} ({datos[0]})?',
            QMessageBox.Yes | QMessageBox.No
        )
        if respuesta == QMessageBox.Yes:
            row = self.lista_docentes.row(item_actual)
            self.lista_docentes.takeItem(row)
            self.guardar_datos()
            self.detalles_text.clear()
            QMessageBox.information(self, 'Éxito', 'Docente eliminado correctamente.')

    def limpiar_formulario(self):
        self.legajo_edit.clear()
        self.nombre_edit.clear()
        self.apellido_edit.clear()
        self.dni_edit.clear()
        self.email_edit.clear()
        self.telefono_edit.clear()
        self.materia_edit.clear()
        self.categoria_combo.setCurrentIndex(0)

        # Si estaba en edición, restaurar el botón
        if self.item_en_edicion is not None:
            self.item_en_edicion = None
            self.btn_agregar.setText("Agregar Docente")
            try:
                self.btn_agregar.clicked.disconnect()
            except TypeError:
                pass
            self.btn_agregar.clicked.connect(self.agregar_docente)

 
    def exportar_datos(self):
        """Exportar datos a CSV con encabezados."""
        archivo, _ = QFileDialog.getSaveFileName(
            self, 'Exportar datos', 'docentes_export.csv', 'Archivos CSV (*.csv)'
        )
        if not archivo:
            return

        try:
            with open(archivo, 'w', encoding='utf-8', newline='') as f:
                writer = csv.writer(f)
                writer.writerow(['legajo', 'nombre', 'apellido', 'dni',
                                 'email', 'telefono', 'materia', 'categoria'])
                for i in range(self.lista_docentes.count()):
                    item = self.lista_docentes.item(i)
                    writer.writerow(item.data(Qt.UserRole))
            QMessageBox.information(self, 'Exportación exitosa',
                                    f'Datos exportados a:\n{archivo}')
        except Exception as e:
            QMessageBox.critical(self, 'Error', f'No se pudo exportar:\n{e}')

    def mostrar_estadisticas(self):
        """Mostrar cantidad de docentes por categoría."""
        categorias = []
        for i in range(self.lista_docentes.count()):
            item = self.lista_docentes.item(i)
            datos = item.data(Qt.UserRole)
            categorias.append(datos[7])
        conteo = Counter(categorias)
        if not conteo:
            QMessageBox.information(self, 'Estadísticas', 'No hay datos para mostrar.')
            return

        texto = "Cantidad por categoría:\n\n"
        for cat in ["Titular", "Asociado", "Adjunto", "Auxiliar", "Interino"]:
            texto += f"- {cat}: {conteo.get(cat, 0)}\n"
        QMessageBox.information(self, 'Estadísticas', texto)


if __name__ == '__main__':
    app = QApplication(sys.argv)
    sistema = SistemaDocentes()
    sistema.show()
    sys.exit(app.exec_())
