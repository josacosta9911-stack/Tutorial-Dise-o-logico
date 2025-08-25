# Tutorial-Diseñologico

[TUTORIAL Uso del toolchain para diseño FPGA.zip](https://github.com/user-attachments/files/21955241/TUTORIAL.open_source_fpga_environment-main.zip)

[TUTORIAL Primer diseño, desde el RTL hasta el bitsream.zip](https://github.com/user-attachments/files/21955242/PROYECTO.TUTORIAL.zip)

### Plantilla para proyectos 

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/10d57b09-8748-4bd5-8d21-85b0a748183e" />

### Clonar el repositorio 

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/1618fb4e-7b58-4d7c-9149-bc8b1bb0c354" />

### Uso de las herremientas 

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/25f5a97b-e59f-48d7-a167-b78026c57a17" />

### Prueba de las herramientas de síntesis, simulación e implementación.

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/a5dce095-ddfd-4d5b-b25b-4c75ee563d26" />

### Ruta al Makefile 

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/174173f3-f1bf-4e64-bc15-36a6f1f63923" />

### Verificacion de los diseños y comandos (make test )

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/90b71ee5-6cfd-4bbf-98a7-467c191ab799" />

### Diagramas de Tiempo (make wv )

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/b00ff955-0b9c-4282-99bf-ce565ef6db31" />

### Verificar la sintaxis y sistetizar los diseños RTL (make synth )

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/42d6f69a-8f2c-4cd7-9f9b-9b71f3920652" />

### Place and router que contiene el diseño en la FPGA ( make pnr )

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/8cc587e8-7451-4a40-b9e6-191ad9c50558" />

### Generar el biststeam ( make biststeam )

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/b79cd242-3f0a-428d-b1b7-f2163eb3bab5" />

###   Cargar el bitsream ( make load )

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/f2b597d4-4b82-4d13-a869-1feb2db6c4fb" />

### Ejecutar todos los conmandos de implementacion fisica ( make all )

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/7749c8ba-9578-4fff-913a-8624140ffe9f" />

### Implementacion en la FPGA

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/ca152925-169e-4b80-8190-85b448ec2ff2" />

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/cecaff01-8abf-45af-96dc-f5590a5ee5a1" />


### Carpeta build del ejemplo lcd.spi 
<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/5bf7bbb4-0a4b-4087-a66f-80ad2da4f8e9" />
<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/8660204d-49d4-44ce-9e3f-5733afa4864e" />





# Primer diseño, desde el RTL hasta el bitstream


### Desing 

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/7f154ea9-0034-435a-a0cb-eae53096f187" />

### Sim 

```
`timescale  1ns/1ps

module module_counter_tb;

    logic clk;
    logic rst;    
    logic [5: 0] count_o;

    module_counter # (10) COUNT (
        .clk (clk),
        .rst (rst),
        .count_o (count_o)
    );

    initial begin
        
    clk = 0;
    rst = 0;

    #30;

    rst = 0; 

    #30;

    rst = 1; 

    #300000
    $finish; 
    end

    always begin 
        clk = ~clk;
        #10
    end

    initial begin 
        $dumpfile("module_counter_tb.vcd");
        $dumpvars(0, module_counter_tb);
    end 
```
endmodule 

### Makefile 
```
#Makefile con todo el flujo de trabajo para GOWIN. Utilizando Yosys, nextpnr, iverilog, gtkwave y openFPGALoader

#FPGA a utilizar... Esto no se debe modificar para efectos del curso.
BOARD  = tangnano9k
FAMILY = GW1N-9C
DEVICE = GW1NR-LV9QN88PC6/I5

#Nombre del proyecto... Acá ponen el nombre que deseen.
PROYECT = counter_demo 

#Fuentes de diseno
SOURCES := $(wildcard ../design/*.v ../design/*.sv) #Todas las fuentes .v o .sv que estan en design
#Si quieren indicarlas una a una, pueden hacerlo como en este ejemplo:
#SOURCES = ../design/module_top_deco_gray.v ../design/module_input_deco_gray.v 

#Fuente de simulacion
#aca va el testbench que quieran simular
TESTBENCH = ../sim/module_counter_tb.sv

# Constraints para el proyecto
#aca va el archivo de constraints del proyecto
CONSTRAINTS = ../constr/module_counter.cst

#el top se indica sin la extension .v, esto hace referencia al nombre que le pusieron al módulo y no al archivo en si.
TOP_DESIGN = module_counter
TOP_TB     = module_counter_tb

#nombre del vcd que va a generar el tb
VCD_FILE = module_counter_tb.vcd


all: synth pnr bitstream load

# Synthesis
synth: ${SOURCES}
	@echo "Ejecutando la sintesis..."
	@yosys -p "read_verilog -sv ${SOURCES}; synth_gowin -top ${TOP_DESIGN} -json ${PROYECT}.json" > synthesis_${BOARD}.log 2>&1 
	@echo "COMPLETADO"

# Place and Route
pnr: ${PROYECT}.json
	@echo "Ejecutando el pnr..."
	@nextpnr-gowin --json ${PROYECT}.json --write ${PROYECT}_pnr.json --freq 27 --device ${DEVICE} --family ${FAMILY} --cst ${CONSTRAINTS} > pnr_${BOARD}.log 2>&1 
	@echo "COMPLETADO"

# Generar el Bitstream
bitstream: ${PROYECT}_pnr.json
	@echo "Generando ${PROYECT}_${BOARD}.fs"
	@gowin_pack -d ${FAMILY} -o ${PROYECT}_${BOARD}.fs ${PROYECT}_pnr.json
	@echo "COMPLETADO"
		
#Generar vcd con icarus verilog
test: ${SOURCES} ${TESTBENCH}
	@iverilog -o ${PROYECT}_test.o -s ${TOP_TB} -g2005-sv ${TESTBENCH} ${SOURCES}
	@vvp ${PROYECT}_test.o 
	
#Visualizar los diagramas de tiempo con GTKWave
wv: ${VCD_FILE}
	gtkwave ${VCD_FILE} 
        
# Cargar el bitstream en la FPGA
load: ${PROYECT}_${BOARD}.fs
	openFPGALoader -b ${BOARD} ${PROYECT}_${BOARD}.fs 

.PHONY: all synth pnr bitstream test wv load
.INTERMEDIATE: ${PROYECT}_pnr.json ${PROYECT}.json
```













