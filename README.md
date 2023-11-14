# Odoo
es un repo para ir guardando mi avance


# -*- coding: utf-8 -*-

from odoo import fields, models, api
from odoo.exceptions import ValidationError

import random

from .nomenclador import CARGA_MODEL_NAME
from datetime import datetime, timedelta


TIPO_BUQUE_MODEL_NAME = "tipo.buque"
TIPO_EMBARQUE_MODEL_NAME = "tipo.embarque"
BUQ_MODEL_NAME = "nom.buque"

class TipoEmbarque(models.Model):
    _name = TIPO_EMBARQUE_MODEL_NAME
    _description = 'Tipo de Embarque'
    

    fecha = fields.Date(string='Fecha')
    no_dm_ti = fields.Integer(string='No. DM Ti')

    # id_del_bl = TcxIntegerField(
    #     "Id del Bl",
    #     readonly=True,
    #     help="Es el identificador del plan de importación.\n Se genera una vez que se guardan los cambios.",
    # )
    # id_del_bl = fields.Char(string='ID del BL', default=lambda self: str(random.randint(10000, 99999)), readonly=True)

    id_del_bl = fields.Char(string='ID del BL', default=lambda self: str(random.randint(1000000000, 9999999999)), readonly=True)
    
    
    tipo_carga = fields.Many2one(CARGA_MODEL_NAME,"Carga",required= True)
    

    
    fecha_lista_presentacion = fields.Date(string='Fecha Lista Presentación')
    fecha_extraccion = fields.Date(string='Fecha de Extracción')
    fecha_levante = fields.Date(string='Fecha de Levante')
    fecha_ent_nacionales = fields.Date(string='Fecha Ent. en Nacionales')
    
    @api.constrains('no_dm_ti')
    def _check_no_dm_ti(self):
        for record in self:
            if record.no_dm_ti <= 0:
                raise ValidationError('El número DM Ti debe ser mayor a cero.')
    
    @api.constrains('fecha_lista_presentacion', 'fecha_extraccion', 'fecha_levante', 'fecha_ent_nacionales')
    def _check_fecha(self):
        for record in self:
            if record.fecha_lista_presentacion and record.fecha_extraccion and record.fecha_levante and record.fecha_ent_nacionales:
                if record.fecha_lista_presentacion > record.fecha_extraccion or record.fecha_extraccion > record.fecha_levante or record.fecha_levante > record.fecha_ent_nacionales:
                    raise ValidationError('Las fechas deben estar en orden cronológico.')


    no_dm_aduana = fields.Char(string='No. DM Aduana')
    tipo_dm = fields.Selection(selection=[('A', 'A'), ('B', 'B'), ('C', 'C')], string='Tipo DM')
    aduana = fields.Selection(selection=[('A', 'A'), ('B', 'B'), ('C', 'C')], string='Aduana')
    fecha_presentacion = fields.Date(string='Fecha de Presentacion') 

    # @api.constrains('fecha_presentacion')
    # def _check_fecha_presentacion(self):
    #     for record in self:
    #         if record.fecha_presentacion and record.fecha_presentacion > fields.Date.today():
    #             raise ValidationError('La fecha de presentación no puede ser posterior a la fecha actual.')
            
   

    tipo_importacion = fields.Boolean(string='Es una importación temporal')
    fecha_real_culminacion_importacion = fields.Date(string='Fecha Real de Culminacion de Importación')
    fecha_plan_culminacion_importacion = fields.Date(string='Fecha Plan de Culminacion de Importación')
    
    # @api.constrains('fecha_plan_culminacion_importacion')
    # def _check_fecha_plan_culminacion_importacion(self):
    #     for record in self:
    #         if record.fecha_plan_culminacion_importacion and record.fecha_plan_culminacion_importacion > fields.Date.today():
    #             raise ValidationError('La fecha de presentación no puede ser posterior a la fecha actual.')
    
 

class TipoBuque(models.Model):
        _name = TIPO_BUQUE_MODEL_NAME
        _description = 'Tipo de Buque'

        

        buque_viaje_fecha = fields.Selection([
        ('buque', 'Buque'),
        ('viaje', 'Viaje'),
        ('fecha_llegada', 'Fecha de Llegada'),
    ], string='Buque/Viaje/Fecha de Llegada')
        
        SELECCION_BUQUE = [('a', 'A'), ('b', 'B'), ('c', 'C')]

        # seleccion_buque=fields.Many2one(BUQ_MODEL_NAME,"Seleccionar Buque",required=True)

        seleccion_buque = fields.Selection(SELECCION_BUQUE, string='Selección de Buque')
        
        @api.depends('seleccion_buque')
        def _compute_eta(self):
            for rec in self:
                if rec.seleccion_buque == 'a':
                    rec.eta = (datetime.now() + timedelta(days=3)).date()
                elif rec.seleccion_buque == 'b':
                    rec.eta = (datetime.now() + timedelta(days=5)).date()
                elif rec.seleccion_buque == 'c':
                    rec.eta = (datetime.now() + timedelta(days=7)).date()
                else:
                    rec.eta = False

        fecha = fields.Date(string='Fecha de Arribo', compute='_compute_fecha', store=True)
        @api.depends('seleccion_buque')
        def _compute_fecha(self):
            for rec in self:
                if rec.seleccion_buque == 'Titanic':
                    rec.fecha = (datetime.now() + timedelta(days=7)).date()
                elif rec.seleccion_buque == 'b':
                    rec.fecha = (datetime.now() + timedelta(days=9)).date()
                elif rec.seleccion_buque == 'c':
                    rec.fecha = (datetime.now() + timedelta(days=11)).date()
                else:
                    rec.fecha = False
        # @api.depends('seleccion_buque')
        # def _compute_arribo(self):
        #     for rec in self:
        #         if rec.seleccion_buque == 'a':
        #             rec.eta = (datetime.now() + timedelta(days=7)).date()
        #         elif rec.seleccion_buque == 'b':
        #             rec.eta = (datetime.now() + timedelta(days=9)).date()
        #         else:
        #             rec.eta = False

        seleccion_bu = fields.Selection(SELECCION_BUQUE, string='Selección de Buque')

        eta = fields.Date(string='Fecha de ETA', compute='_compute_eta', store=True)
        bl = fields.Date(string='Fecha del BL')

        @api.constrains
        def ver_todos_buques(self):
            buques = self.search([])
            for buque in buques:
                print("Buque/Viaje/Fecha de Llegada:", buque.buque_viaje_fecha)
                print("Selección de Buque:", buque.seleccion_buque)
                print("ETA:", buque.eta)
                print("BL:", buque.bl)
                print("------")
        campo_sin_nombre = fields.Char(string='', required=True)
        campo_sin_nomb = fields.Char(string='', required=True)
        
        notificar = fields.Char(string="Notificar", size=100)
        fecha_entrega = fields.Date(string="Fecha de Entrega")
        consignado = fields.Char(string="Consignado", default="TECNOTEX")
        
        naviera = fields.Selection([
        ('a', 'A'),
        ('b', 'B'),
        ('c', 'C')
         ], string='Naviera')

        transitorio = fields.Selection([
        ('a', 'A'),
        ('b', 'B'),
        ('c', 'C')
        ], string='Transitorio')
        manifesto = fields.Text(string='Manifesto')
        embarcador = fields.Char(string='Embarcador')

        

       

        trasbordo = fields.Selection([
        ('buque', 'Buque'),
        ('viaje', 'Viaje'),
        ('fecha_llegada', 'Fecha de Llegada'),
        ], string='Trasbordo')

        origen = fields.Selection([
        ('cuba', 'Cuba'),
        ('mx', 'MX'),
        ('colombia', 'Colombia'),
        ], string='Origen')
        
        destino = fields.Selection([
        ('cuba', 'Cuba'),
        ('mx', 'MX'),
        ('colombia', 'Colombia'),
        ], string='Destino')

        copia = fields.Date(string="Copia del Original")
        original = fields.Date(string="Original")
        
class NomBuque(models.Model):
    _name = BUQ_MODEL_NAME
    _description = 'Buque'
    _rec_name = "nombre"

    nombre = fields.Char(string="Nombre", required=True)
    
