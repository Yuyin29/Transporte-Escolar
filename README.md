# transporte_escolar.py
import tkinter as tk
from tkinter import ttk, messagebox
from abc import ABC, abstractmethod
import mariadb
from datetime import time

# CONFIG: Ajusta credenciales
DB_CONFIG = {
    "user": "root",
    "password": "",
    "host": "127.0.0.1",
    "database": "transportes"
}

# CAPA DE DATOS: DatabaseManager
class DatabaseManager:
    def __init__(self, config):
        self.config = config

    def connect(self):
        try:
            conn = mariadb.connect(**self.config)
            return conn
        except mariadb.Error as e:
            messagebox.showerror("DB Error", f"No se pudo conectar a la base de datos: {e}")
            return None

    # Métodos para choferes
    def insert_chofer(self, nombre, a_paterno, a_materno, licencia, telefono, ruta_id=None, autobus_id=None):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute(
                "INSERT INTO choferes (nombre, a_paterno, a_materno, licencia, telefono, ruta_id, autobus_id) "
                "VALUES (%s,%s,%s,%s,%s,%s,%s)",
                (nombre, a_paterno, a_materno, licencia, telefono, ruta_id, autobus_id)
            )
            conn.commit()
            return True
        except mariadb.Error as e:
            messagebox.showerror("Error insertar chofer", str(e))
            return False
        finally:
            conn.close()

    def get_choferes(self):
        conn = self.connect()
        if not conn: return []
        try:
            cur = conn.cursor()
            cur.execute("SELECT id, nombre, a_paterno, a_materno, licencia, telefono, ruta_id, autobus_id FROM choferes")
            return cur.fetchall()
        except mariadb.Error as e:
            messagebox.showerror("Error consultar choferes", str(e))
            return []
        finally:
            conn.close()

    def update_chofer(self, id_, nombre, a_paterno, a_materno, licencia, telefono, ruta_id, autobus_id):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute(
                "UPDATE choferes SET nombre=%s, a_paterno=%s, a_materno=%s, licencia=%s, telefono=%s, ruta_id=%s, autobus_id=%s WHERE id=%s",
                (nombre, a_paterno, a_materno, licencia, telefono, ruta_id, autobus_id, id_)
            )
            conn.commit()
            return cur.rowcount > 0
        except mariadb.Error as e:
            messagebox.showerror("Error actualizar chofer", str(e))
            return False
        finally:
            conn.close()

    def delete_chofer(self, id_):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute("DELETE FROM choferes WHERE id=%s", (id_,))
            conn.commit()
            return cur.rowcount > 0
        except mariadb.Error as e:
            messagebox.showerror("Error eliminar chofer", str(e))
            return False
        finally:
            conn.close()

    # Métodos para estudiantes
    def insert_estudiante(self, nombre, a_paterno, a_materno, grado, grupo, ruta_id=None):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute(
                "INSERT INTO estudiantes (nombre, a_paterno, a_materno, grado, grupo, ruta_id) VALUES (%s,%s,%s,%s,%s,%s)",
                (nombre, a_paterno, a_materno, grado, grupo, ruta_id)
            )
            conn.commit()
            return True
        except mariadb.Error as e:
            messagebox.showerror("Error insertar estudiante", str(e))
            return False
        finally:
            conn.close()

    def get_estudiantes(self):
        conn = self.connect()
        if not conn: return []
        try:
            cur = conn.cursor()
            cur.execute("SELECT id, nombre, a_paterno, a_materno, grado, grupo, ruta_id FROM estudiantes")
            return cur.fetchall()
        except mariadb.Error as e:
            messagebox.showerror("Error consultar estudiantes", str(e))
            return []
        finally:
            conn.close()

    def update_estudiante(self, id_, nombre, a_paterno, a_materno, grado, grupo, ruta_id):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute(
                "UPDATE estudiantes SET nombre=%s, a_paterno=%s, a_materno=%s, grado=%s, grupo=%s, ruta_id=%s WHERE id=%s",
                (nombre, a_paterno, a_materno, grado, grupo, ruta_id, id_)
            )
            conn.commit()
            return cur.rowcount > 0
        except mariadb.Error as e:
            messagebox.showerror("Error actualizar estudiante", str(e))
            return False
        finally:
            conn.close()

    def delete_estudiante(self, id_):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute("DELETE FROM estudiantes WHERE id=%s", (id_,))
            conn.commit()
            return cur.rowcount > 0
        except mariadb.Error as e:
            messagebox.showerror("Error eliminar estudiante", str(e))
            return False
        finally:
            conn.close()

    # Métodos para rutas/autobuses/horarios (básicos)
    def insert_ruta(self, nombre, origen, destino):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute("INSERT INTO rutas (nombre, origen, destino) VALUES (%s,%s,%s)", (nombre, origen, destino))
            conn.commit()
            return True
        except mariadb.Error as e:
            messagebox.showerror("Error insertar ruta", str(e))
            return False
        finally:
            conn.close()

    def get_rutas(self):
        conn = self.connect()
        if not conn: return []
        try:
            cur = conn.cursor()
            cur.execute("SELECT id, nombre, origen, destino FROM rutas")
            return cur.fetchall()
        except mariadb.Error as e:
            messagebox.showerror("Error consultar rutas", str(e))
            return []
        finally:
            conn.close()

    def insert_autobus(self, placa, capacidad, ruta_id=None):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute("INSERT INTO autobuses (placa, capacidad, ruta_id) VALUES (%s,%s,%s)", (placa, capacidad, ruta_id))
            conn.commit()
            return True
        except mariadb.Error as e:
            messagebox.showerror("Error insertar autobús", str(e))
            return False
        finally:
            conn.close()

    def get_autobuses(self):
        conn = self.connect()
        if not conn: return []
        try:
            cur = conn.cursor()
            cur.execute("SELECT id, placa, capacidad, ruta_id FROM autobuses")
            return cur.fetchall()
        except mariadb.Error as e:
            messagebox.showerror("Error consultar autobuses", str(e))
            return []
        finally:
            conn.close()

    def insert_horario(self, ruta_id, dia, hora_salida, hora_llegada):
        conn = self.connect()
        if not conn: return False
        try:
            cur = conn.cursor()
            cur.execute("INSERT INTO horarios (ruta_id, dia, hora_salida, hora_llegada) VALUES (%s,%s,%s,%s)",
                        (ruta_id, dia, hora_salida, hora_llegada))
            conn.commit()
            return True
        except mariadb.Error as e:
            messagebox.showerror("Error insertar horario", str(e))
            return False
        finally:
            conn.close()

    def get_horarios(self):
        conn = self.connect()
        if not conn: return []
        try:
            cur = conn.cursor()
            cur.execute("SELECT id, ruta_id, dia, hora_salida, hora_llegada FROM horarios")
            return cur.fetchall()
        except mariadb.Error as e:
            messagebox.showerror("Error consultar horarios", str(e))
            return []
        finally:
            conn.close()

# CAPA DE DOMINIO
class Persona(ABC):
    def __init__(self, nombre, a_paterno, a_materno):
        self._nombre = nombre
        self._a_paterno = a_paterno
        self._a_materno = a_materno

    @abstractmethod
    def mostrar_info(self):
        pass

class Chofer(Persona):
    def __init__(self, nombre, a_paterno, a_materno, licencia, telefono, ruta_id=None, autobus_id=None):
        super().__init__(nombre, a_paterno, a_materno)
        self.licencia = licencia
        self.telefono = telefono
        self.ruta_id = ruta_id
        self.autobus_id = autobus_id

    def mostrar_info(self):
        return f"{self._nombre} {self._a_paterno} {self._a_materno} - Lic: {self.licencia}"

class Estudiante(Persona):
    def __init__(self, nombre, a_paterno, a_materno, grado, grupo, ruta_id=None):
        super().__init__(nombre, a_paterno, a_materno)
        self.grado = grado
        self.grupo = grupo
        self.ruta_id = ruta_id

    def mostrar_info(self):
        return f"{self._nombre} {self._a_paterno} {self._a_materno} - {self.grado}{self.grupo}"

class Ruta:
    def __init__(self, nombre, origen, destino):
        self.nombre = nombre
        self.origen = origen
        self.destino = destino

class Autobus:
    def __init__(self, placa, capacidad, ruta_id=None):
        self.placa = placa
        self.capacidad = capacidad
        self.ruta_id = ruta_id

class Horario:
    def __init__(self, ruta_id, dia, hora_salida, hora_llegada):
        self.ruta_id = ruta_id
        self.dia = dia
        self.hora_salida = hora_salida
        self.hora_llegada = hora_llegada

# INTERFAZ GRÁFICA (Tkinter)
class App:
    def __init__(self, root, db: DatabaseManager):
        self.root = root
        self.db = db
        root.title("Sistema de Control de Transporte Escolar")
        root.geometry("900x600")

        notebook = ttk.Notebook(root)
        notebook.pack(fill="both", expand=True, padx=10, pady=10)

        # Pestañas
        self.tab_choferes = ttk.Frame(notebook)
        self.tab_estudiantes = ttk.Frame(notebook)
        self.tab_rutas = ttk.Frame(notebook)
        self.tab_autobuses = ttk.Frame(notebook)
        self.tab_horarios = ttk.Frame(notebook)

        notebook.add(self.tab_choferes, text="Choferes")
        notebook.add(self.tab_estudiantes, text="Estudiantes")
        notebook.add(self.tab_rutas, text="Rutas")
        notebook.add(self.tab_autobuses, text="Autobuses")
        notebook.add(self.tab_horarios, text="Horarios")

        self.build_choferes_tab()
        self.build_estudiantes_tab()
        self.build_rutas_tab()
        self.build_autobuses_tab()
        self.build_horarios_tab()

    # ---------- Choferes ----------
    def build_choferes_tab(self):
        f = self.tab_choferes
        labels = ["ID (para editar/eliminar):", "Nombre", "Apellido paterno", "Apellido materno", "Licencia", "Teléfono", "Ruta ID", "Autobús ID"]
        self.chofer_entries = {}
        for i, txt in enumerate(labels):
            ttk.Label(f, text=txt).grid(row=i, column=0, sticky="w", padx=5, pady=4)
            ent = ttk.Entry(f, width=30)
            ent.grid(row=i, column=1, padx=5, pady=4, sticky="w")
            self.chofer_entries[txt] = ent

        btns = [("Insertar", self.insertar_chofer), ("Actualizar", self.actualizar_chofer), ("Eliminar", self.eliminar_chofer), ("Refrescar lista", self.refrescar_choferes)]
        for i, (txt, cmd) in enumerate(btns):
            ttk.Button(f, text=txt, command=cmd).grid(row=0, column=2 + i, padx=5, pady=4)

        # Lista
        self.list_choferes = tk.Listbox(f, width=90, height=18)
        self.list_choferes.grid(row=8, column=0, columnspan=4, padx=5, pady=8)
        self.refrescar_choferes()

    def insertar_chofer(self):
        e = self.chofer_entries
        nombre = e["Nombre"].get().strip()
        ap = e["Apellido paterno"].get().strip()
        am = e["Apellido materno"].get().strip()
        licencia = e["Licencia"].get().strip()
        tel = e["Teléfono"].get().strip()
        ruta_id = e["Ruta ID"].get().strip() or None
        autobus_id = e["Autobús ID"].get().strip() or None

        if not (nombre and ap):
            messagebox.showwarning("Faltan datos", "Nombre y apellido paterno son requeridos.")
            return
        # convertir ids si existen
        ruta_id = int(ruta_id) if ruta_id and ruta_id.isdigit() else None
        autobus_id = int(autobus_id) if autobus_id and autobus_id.isdigit() else None

        ok = self.db.insert_chofer(nombre, ap, am, licencia, tel, ruta_id, autobus_id)
        if ok:
            messagebox.showinfo("Éxito", "Chofer insertado.")
            self.limpiar_choferes_campos()
            self.refrescar_choferes()

    def refrescar_choferes(self):
        self.list_choferes.delete(0, tk.END)
        rows = self.db.get_choferes()
        for r in rows:
            id_, nombre, ap, am, lic, tel, ruta_id, autobus_id = r
            self.list_choferes.insert(tk.END, f"ID:{id_} - {nombre} {ap} {am} | Lic:{lic} | Tel:{tel} | Ruta:{ruta_id} | Bus:{autobus_id}")

    def actualizar_chofer(self):
        e = self.chofer_entries
        id_ = e["ID (para editar/eliminar):"].get().strip()
        if not id_.isdigit():
            messagebox.showwarning("ID inválido", "Ingresa un ID válido para actualizar.")
            return
        id_ = int(id_)
        nombre = e["Nombre"].get().strip()
        ap = e["Apellido paterno"].get().strip()
        am = e["Apellido materno"].get().strip()
        licencia = e["Licencia"].get().strip()
        tel = e["Teléfono"].get().strip()
        ruta_id = e["Ruta ID"].get().strip() or None
        autobus_id = e["Autobús ID"].get().strip() or None
        ruta_id = int(ruta_id) if ruta_id and ruta_id.isdigit() else None
        autobus_id = int(autobus_id) if autobus_id and autobus_id.isdigit() else None
        ok = self.db.update_chofer(id_, nombre, ap, am, licencia, tel, ruta_id, autobus_id)
        if ok:
            messagebox.showinfo("Éxito", "Chofer actualizado.")
            self.refrescar_choferes()
        else:
            messagebox.showinfo("Sin cambios", "No se actualizó (revisa el ID).")

    def eliminar_chofer(self):
        id_ = self.chofer_entries["ID (para editar/eliminar):"].get().strip()
        if not id_.isdigit():
            messagebox.showwarning("ID inválido", "Ingresa un ID válido para eliminar.")
            return
        if not messagebox.askyesno("Confirmar", "¿Eliminar chofer?"):
            return
        ok = self.db.delete_chofer(int(id_))
        if ok:
            messagebox.showinfo("Éxito", "Chofer eliminado.")
            self.refrescar_choferes()
            self.limpiar_choferes_campos()
        else:
            messagebox.showinfo("Sin cambios", "No se eliminó (revisa el ID).")

    def limpiar_choferes_campos(self):
        for ent in self.chofer_entries.values():
            ent.delete(0, tk.END)

    # ---------- Estudiantes ----------
    def build_estudiantes_tab(self):
        f = self.tab_estudiantes
        labels = ["ID (editar/eliminar):", "Nombre", "Apellido paterno", "Apellido materno", "Grado", "Grupo", "Ruta ID"]
        self.est_entries = {}
        for i, txt in enumerate(labels):
            ttk.Label(f, text=txt).grid(row=i, column=0, sticky="w", padx=5, pady=4)
            ent = ttk.Entry(f, width=30)
            ent.grid(row=i, column=1, padx=5, pady=4, sticky="w")
            self.est_entries[txt] = ent

        ttk.Button(f, text="Insertar", command=self.insertar_estudiante).grid(row=0, column=2, padx=5)
        ttk.Button(f, text="Actualizar", command=self.actualizar_estudiante).grid(row=0, column=3, padx=5)
        ttk.Button(f, text="Eliminar", command=self.eliminar_estudiante).grid(row=0, column=4, padx=5)
        ttk.Button(f, text="Refrescar lista", command=self.refrescar_estudiantes).grid(row=0, column=5, padx=5)

        self.list_est = tk.Listbox(f, width=90, height=18)
        self.list_est.grid(row=8, column=0, columnspan=6, padx=5, pady=8)
        self.refrescar_estudiantes()

    def insertar_estudiante(self):
        e = self.est_entries
        nombre = e["Nombre"].get().strip()
        ap = e["Apellido paterno"].get().strip()
        am = e["Apellido materno"].get().strip()
        grado = e["Grado"].get().strip()
        grupo = e["Grupo"].get().strip()
        ruta_id = e["Ruta ID"].get().strip() or None
        ruta_id = int(ruta_id) if ruta_id and ruta_id.isdigit() else None

        if not (nombre and ap):
            messagebox.showwarning("Faltan datos", "Nombre y apellido paterno son requeridos.")
            return

        ok = self.db.insert_estudiante(nombre, ap, am, grado, grupo, ruta_id)
        if ok:
            messagebox.showinfo("Éxito", "Estudiante insertado.")
            self.limpiar_est_campos()
            self.refrescar_estudiantes()

    def refrescar_estudiantes(self):
        self.list_est.delete(0, tk.END)
        rows = self.db.get_estudiantes()
        for r in rows:
            id_, nombre, ap, am, grado, grupo, ruta_id = r
            self.list_est.insert(tk.END, f"ID:{id_} - {nombre} {ap} {am} | {grado}{grupo} | Ruta:{ruta_id}")

    def actualizar_estudiante(self):
        e = self.est_entries
        id_ = e["ID (editar/eliminar):"].get().strip()
        if not id_.isdigit():
            messagebox.showwarning("ID inválido", "Ingresa un ID válido.")
            return
        id_ = int(id_)
        nombre = e["Nombre"].get().strip()
        ap = e["Apellido paterno"].get().strip()
        am = e["Apellido materno"].get().strip()
        grado = e["Grado"].get().strip()
        grupo = e["Grupo"].get().strip()
        ruta_id = e["Ruta ID"].get().strip() or None
        ruta_id = int(ruta_id) if ruta_id and ruta_id.isdigit() else None
        ok = self.db.update_estudiante(id_, nombre, ap, am, grado, grupo, ruta_id)
        if ok:
            messagebox.showinfo("Éxito", "Estudiante actualizado.")
            self.refrescar_estudiantes()
        else:
            messagebox.showinfo("Sin cambios", "No se actualizó (revisa el ID).")

    def eliminar_estudiante(self):
        id_ = self.est_entries["ID (editar/eliminar):"].get().strip()
        if not id_.isdigit():
            messagebox.showwarning("ID inválido", "Ingresa un ID válido.")
            return
        if not messagebox.askyesno("Confirmar", "¿Eliminar estudiante?"):
            return
        ok = self.db.delete_estudiante(int(id_))
        if ok:
            messagebox.showinfo("Éxito", "Estudiante eliminado.")
            self.refrescar_estudiantes()
            self.limpiar_est_campos()
        else:
            messagebox.showinfo("Sin cambios", "No se eliminó (revisa el ID).")

    def limpiar_est_campos(self):
        for ent in self.est_entries.values():
            ent.delete(0, tk.END)

    # ---------- Rutas ----------
    def build_rutas_tab(self):
        f = self.tab_rutas
        ttk.Label(f, text="Nombre ruta:").grid(row=0, column=0, padx=5, pady=4, sticky="w")
        self.ruta_nombre = ttk.Entry(f, width=30); self.ruta_nombre.grid(row=0, column=1)
        ttk.Label(f, text="Origen:").grid(row=1, column=0, padx=5, pady=4, sticky="w")
        self.ruta_origen = ttk.Entry(f, width=30); self.ruta_origen.grid(row=1, column=1)
        ttk.Label(f, text="Destino:").grid(row=2, column=0, padx=5, pady=4, sticky="w")
        self.ruta_destino = ttk.Entry(f, width=30); self.ruta_destino.grid(row=2, column=1)

        ttk.Button(f, text="Insertar ruta", command=self.insertar_ruta).grid(row=0, column=2, padx=5)
        ttk.Button(f, text="Refrescar rutas", command=self.refrescar_rutas).grid(row=0, column=3, padx=5)

        self.list_rutas = tk.Listbox(f, width=90, height=18)
        self.list_rutas.grid(row=5, column=0, columnspan=4, padx=5, pady=8)
        self.refrescar_rutas()

    def insertar_ruta(self):
        n = self.ruta_nombre.get().strip()
        o = self.ruta_origen.get().strip()
        d = self.ruta_destino.get().strip()
        if not n:
            messagebox.showwarning("Faltan datos", "El nombre de ruta es requerido.")
            return
        ok = self.db.insert_ruta(n, o, d)
        if ok:
            messagebox.showinfo("Éxito", "Ruta creada.")
            self.ruta_nombre.delete(0, tk.END); self.ruta_origen.delete(0, tk.END); self.ruta_destino.delete(0, tk.END)
            self.refrescar_rutas()

    def refrescar_rutas(self):
        self.list_rutas.delete(0, tk.END)
        rows = self.db.get_rutas()
        for r in rows:
            id_, nombre, origen, destino = r
            self.list_rutas.insert(tk.END, f"ID:{id_} - {nombre} | {origen} -> {destino}")

    # ---------- Autobuses ----------
    def build_autobuses_tab(self):
        f = self.tab_autobuses
        ttk.Label(f, text="Placa:").grid(row=0, column=0, padx=5, pady=4, sticky="w")
        self.bus_placa = ttk.Entry(f, width=20); self.bus_placa.grid(row=0, column=1)
        ttk.Label(f, text="Capacidad:").grid(row=1, column=0, padx=5, pady=4, sticky="w")
        self.bus_cap = ttk.Entry(f, width=10); self.bus_cap.grid(row=1, column=1)
        ttk.Label(f, text="Ruta ID:").grid(row=2, column=0, padx=5, pady=4, sticky="w")
        self.bus_ruta = ttk.Entry(f, width=10); self.bus_ruta.grid(row=2, column=1)

        ttk.Button(f, text="Insertar autobús", command=self.insertar_autobus).grid(row=0, column=2, padx=5)
        ttk.Button(f, text="Refrescar autobuses", command=self.refrescar_autobuses).grid(row=0, column=3, padx=5)

        self.list_buses = tk.Listbox(f, width=90, height=18)
        self.list_buses.grid(row=5, column=0, columnspan=4, padx=5, pady=8)
        self.refrescar_autobuses()

    def insertar_autobus(self):
        placa = self.bus_placa.get().strip()
        cap = self.bus_cap.get().strip()
        ruta_id = self.bus_ruta.get().strip() or None
        if not placa:
            messagebox.showwarning("Faltan datos", "Placa requerida.")
            return
        cap = int(cap) if cap.isdigit() else None
        ruta_id = int(ruta_id) if ruta_id and ruta_id.isdigit() else None
        ok = self.db.insert_autobus(placa, cap, ruta_id)
        if ok:
            messagebox.showinfo("Éxito", "Autobús creado.")
            self.bus_placa.delete(0, tk.END); self.bus_cap.delete(0, tk.END); self.bus_ruta.delete(0, tk.END)
            self.refrescar_autobuses()

    def refrescar_autobuses(self):
        self.list_buses.delete(0, tk.END)
        rows = self.db.get_autobuses()
        for r in rows:
            id_, placa, cap, ruta_id = r
            self.list_buses.insert(tk.END, f"ID:{id_} - {placa} | Cap:{cap} | Ruta:{ruta_id}")

    # ---------- Horarios ----------
    def build_horarios_tab(self):
        f = self.tab_horarios
        ttk.Label(f, text="Ruta ID:").grid(row=0, column=0, padx=5, pady=4, sticky="w")
        self.h_ruta = ttk.Entry(f, width=10); self.h_ruta.grid(row=0, column=1)
        ttk.Label(f, text="Día:").grid(row=1, column=0, padx=5, pady=4, sticky="w")
        self.h_dia = ttk.Entry(f, width=20); self.h_dia.grid(row=1, column=1)
        ttk.Label(f, text="Hora salida (HH:MM):").grid(row=2, column=0, padx=5, pady=4, sticky="w")
        self.h_salida = ttk.Entry(f, width=10); self.h_salida.grid(row=2, column=1)
        ttk.Label(f, text="Hora llegada (HH:MM):").grid(row=3, column=0, padx=5, pady=4, sticky="w")
        self.h_llegada = ttk.Entry(f, width=10); self.h_llegada.grid(row=3, column=1)

        ttk.Button(f, text="Insertar horario", command=self.insertar_horario).grid(row=0, column=2, padx=5)
        ttk.Button(f, text="Refrescar horarios", command=self.refrescar_horarios).grid(row=0, column=3, padx=5)

        self.list_hor = tk.Listbox(f, width=90, height=18)
        self.list_hor.grid(row=5, column=0, columnspan=4, padx=5, pady=8)
        self.refrescar_horarios()

    def insertar_horario(self):
        ruta_id = self.h_ruta.get().strip()
        dia = self.h_dia.get().strip()
        s = self.h_salida.get().strip()
        l = self.h_llegada.get().strip()
        if not ruta_id.isdigit():
            messagebox.showwarning("Datos", "Ruta ID debe ser numérico.")
            return
        try:
            # validar formato HH:MM
            sh, sm = map(int, s.split(":"))
            lh, lm = map(int, l.split(":"))
            s_time = f"{sh:02d}:{sm:02d}:00"
            l_time = f"{lh:02d}:{lm:02d}:00"
        except Exception:
            messagebox.showwarning("Formato hora", "Usa HH:MM en horas.")
            return
        ok = self.db.insert_horario(int(ruta_id), dia, s_time, l_time)
        if ok:
            messagebox.showinfo("Éxito", "Horario creado.")
            self.h_ruta.delete(0, tk.END); self.h_dia.delete(0, tk.END); self.h_salida.delete(0, tk.END); self.h_llegada.delete(0, tk.END)
            self.refrescar_horarios()

    def refrescar_horarios(self):
        self.list_hor.delete(0, tk.END)
        rows = self.db.get_horarios()
        for r in rows:
            id_, ruta_id, dia, hs, hl = r
            self.list_hor.insert(tk.END, f"ID:{id_} | Ruta:{ruta_id} | {dia} | {hs} -> {hl}")


if __name__ == "__main__":
    db = DatabaseManager(DB_CONFIG)
    root = tk.Tk()
    app = App(root, db)
    root.mainloop()
