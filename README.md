# NS-3

-Daniel Ayala

-Cristian Bello

-Samuel Chaves

-Kevin Guevara 

## SIMULACIÓN 

En la siguiente imagen se puede apreciar la union de los nodos, donde el nodo n0 le envia datos al nodo n1 y n2, y el nodo 1 tambien envia datos al n2.

![image](https://github.com/user-attachments/assets/2c6a1fc9-0de5-4105-a62f-3578a2268481)

## CODIGO
```do
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"
#include "ns3/netanim-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("ThreeNodeTopology");

int main(int argc, char* argv[]) {
    CommandLine cmd(__FILE__);
    cmd.Parse(argc, argv);

    Time::SetResolution(Time::NS);
    LogComponentEnable("UdpEchoClientApplication", LOG_LEVEL_INFO);
    LogComponentEnable("UdpEchoServerApplication", LOG_LEVEL_INFO);

    NodeContainer nodes;
    nodes.Create(3);

    PointToPointHelper pointToPoint;
    pointToPoint.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    pointToPoint.SetChannelAttribute("Delay", StringValue("2ms"));

    NetDeviceContainer devices01, devices02, devices12;
    devices01 = pointToPoint.Install(nodes.Get(0), nodes.Get(1));
    devices02 = pointToPoint.Install(nodes.Get(0), nodes.Get(2));
    devices12 = pointToPoint.Install(nodes.Get(1), nodes.Get(2));

    InternetStackHelper stack;
    stack.Install(nodes);

    Ipv4AddressHelper address;
    
    address.SetBase("10.1.1.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces01 = address.Assign(devices01);

    address.SetBase("10.1.2.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces02 = address.Assign(devices02);

    address.SetBase("10.1.3.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces12 = address.Assign(devices12);

    UdpEchoServerHelper echoServer(9);
    
    ApplicationContainer serverApps1 = echoServer.Install(nodes.Get(1));
    serverApps1.Start(Seconds(1.0));
    serverApps1.Stop(Seconds(10.0));
    
    ApplicationContainer serverApps2 = echoServer.Install(nodes.Get(2));
    serverApps2.Start(Seconds(1.0));
    serverApps2.Stop(Seconds(10.0));

    UdpEchoClientHelper echoClient1(interfaces01.GetAddress(1), 9);
    echoClient1.SetAttribute("MaxPackets", UintegerValue(3));
    echoClient1.SetAttribute("Interval", TimeValue(Seconds(1.0)));
    echoClient1.SetAttribute("PacketSize", UintegerValue(1024));

    UdpEchoClientHelper echoClient2(interfaces02.GetAddress(1), 9);
    echoClient2.SetAttribute("MaxPackets", UintegerValue(3));
    echoClient2.SetAttribute("Interval", TimeValue(Seconds(1.0)));
    echoClient2.SetAttribute("PacketSize", UintegerValue(1024));

    UdpEchoClientHelper echoClient3(interfaces12.GetAddress(1), 9);
    echoClient3.SetAttribute("MaxPackets", UintegerValue(3));
    echoClient3.SetAttribute("Interval", TimeValue(Seconds(1.0)));
    echoClient3.SetAttribute("PacketSize", UintegerValue(1024));

    ApplicationContainer clientApps1 = echoClient1.Install(nodes.Get(0));
    clientApps1.Start(Seconds(2.0));
    clientApps1.Stop(Seconds(10.0));

    ApplicationContainer clientApps2 = echoClient2.Install(nodes.Get(0));
    clientApps2.Start(Seconds(2.0));
    clientApps2.Stop(Seconds(10.0));

    ApplicationContainer clientApps3 = echoClient3.Install(nodes.Get(1));
    clientApps3.Start(Seconds(3.0));
    clientApps3.Stop(Seconds(10.0));

    AnimationInterface anim("three_node_topology.xml");
    anim.SetConstantPosition(nodes.Get(0), 10, 20);
    anim.SetConstantPosition(nodes.Get(1), 30, 20);
    anim.SetConstantPosition(nodes.Get(2), 50, 20);

    Simulator::Run();
    Simulator::Destroy();
    return 0;
}

```

## ANALISIS TCP
(Archivos pcap subidos en el github)
```do
Flow ID: 1 (10.1.1.2 -> 10.1.1.1)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 seconds
  Throughput: 84.9527 kbps
Flow ID: 2 (10.1.1.1 -> 10.1.1.2)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 seconds
  Throughput: 84.9527 kbps
Flow ID: 3 (10.1.2.2 -> 10.1.2.1)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 seconds
  Throughput: 84.9527 kbps
Flow ID: 4 (10.1.2.1 -> 10.1.2.2)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 seconds
  Throughput: 84.9527 kbps
  ``` 

Los resultados muestra que todos los flujos de datos están funcionando de manera óptima en términos de entrega de paquetes y rendimiento de la red:

Entrega de paquetes:

Todos los flujos tienen 100 paquetes transmitidos (Tx) y 100 paquetes recibidos (Rx).
No hay paquetes perdidos en ninguno de los flujos (Lost Packets = 0).
La tasa de pérdida de paquetes (Packet Loss Ratio) es 0%, lo que indica que no hubo congestión ni problemas de transmisión.
Rendimiento de la red:

El retraso promedio (Average Delay) es 0.0066864 segundos (6.69 ms), lo cual es un valor bastante bajo y adecuado para la mayoría de aplicaciones en red.
El rendimiento (Throughput) es de 84.9527 kbps en todos los flujos, lo que sugiere que la red está proporcionando un ancho de banda uniforme.

Simetría en la comunicación:

Hay pares de flujos bidireccionales (ejemplo: Flow ID 1 y 2, Flow ID 3 y 4), lo que indica que la comunicación entre los nodos es equilibrada.

Otro forma de resultado:

Si se quisiera ver otro resultado tocaria cambiar el tamaño del ancho de banda para que algunos datos tengan problemas en enviarse produciendo asi la perdida de paquetes y que el retraso cambie, generando asi que el rendimiento del envio de datos sea menor. 

Conclusión:

- No hay pérdidas ni problemas de transmisión (Dependiendo del ancho de banda).
  
- El rendimiento y el retardo son consistentes en todos los flujos
  
- El tráfico es simétrico y estable.

##GRAFICA python

Se puede ver en la grafica que al ser constante y no haber perdida de paquetes la grafica es lineal.

![image](https://github.com/user-attachments/assets/88c27b40-9f48-4611-87ba-8b628dba94be)

-Codigo
 ```do
import matplotlib.pyplot as plt
import numpy as np

# Datos que proporcionaste
flows = ['Flow 1', 'Flow 2', 'Flow 3', 'Flow 4']
throughput = [84.9527, 84.9527, 84.9527, 84.9527]  # kbps
delay = [0.0066864, 0.0066864, 0.0066864, 0.0066864]  # en segundos
packet_loss = [0, 0, 0, 0]  # %

# Crear la figura y los ejes
fig, ax1 = plt.subplots(figsize=(10, 6))

# Graficar el Throughput
color = 'tab:blue'
ax1.set_xlabel('Flujos')
ax1.set_ylabel('Throughput (kbps)', color=color)
ax1.bar(flows, throughput, color=color, alpha=0.6, label="Throughput (kbps)")
ax1.tick_params(axis='y', labelcolor=color)

# Crear un segundo eje para el Delay y el Packet Loss
ax2 = ax1.twinx()
color = 'tab:red'
ax2.set_ylabel('Delay (s) / Packet Loss (%)', color=color)
ax2.plot(flows, delay, color='tab:red', marker='o', label="Delay (s)")
ax2.plot(flows, packet_loss, color='tab:green', marker='s', label="Packet Loss (%)")
ax2.tick_params(axis='y', labelcolor=color)

# Añadir título y leyenda
fig.tight_layout()  # Ajusta el diseño
plt.title('Gráfica de Desempeño de Flujos de Red')

# Mostrar leyendas
ax1.legend(loc='upper left')
ax2.legend(loc='upper right')

# Mostrar la gráfica
plt.show()
```
-Grafica con "problemas": Esto refleja una red con congestionamiento, variabilidad en el rendimiento y pérdida de paquetes en ciertos flujos.

![image](https://github.com/user-attachments/assets/d10a854b-9a3d-4c44-8af0-ebb6754426bc)



