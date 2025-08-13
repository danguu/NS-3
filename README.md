````markdown
# Simulación de Red con NS-3

Integrantes del equipo:
- Daniel Ayala
- Cristian Bello
- Samuel Chaves
- Kevin Guevara

## 1. Descripción General

Este proyecto implementa una topología de tres nodos utilizando NS-3.  
El nodo `n0` envía datos a `n1` y `n2`, mientras que el nodo `n1` también envía datos al nodo `n2`.  
Se incluyen análisis de rendimiento de red y visualización gráfica de los resultados.

Topología simulada:
````
 
![image](https://github.com/user-attachments/assets/2c6a1fc9-0de5-4105-a62f-3578a2268481)

``` 
## 2. Código de Simulación (C++ con NS-3)

```cpp
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
````
## 3. Análisis TCP

> Archivos `.pcap` disponibles en el repositorio.

```txt
Flow ID: 1 (10.1.1.2 -> 10.1.1.1)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 s
  Throughput: 84.9527 kbps

Flow ID: 2 (10.1.1.1 -> 10.1.1.2)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 s
  Throughput: 84.9527 kbps

Flow ID: 3 (10.1.2.2 -> 10.1.2.1)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 s
  Throughput: 84.9527 kbps

Flow ID: 4 (10.1.2.1 -> 10.1.2.2)
  Tx Packets: 100
  Rx Packets: 100
  Lost Packets: 0
  Packet Loss Ratio: 0%
  Average Delay: 0.0066864 s
  Throughput: 84.9527 kbps
```

Interpretación:

* **Entrega de paquetes:** 100% de entrega, sin pérdidas.
* **Rendimiento:** `84.95 kbps` en todos los flujos.
* **Retardo promedio:** `6.69 ms`, óptimo para la mayoría de aplicaciones.
* **Simetría:** comunicación bidireccional con rendimiento idéntico en ambas direcciones.

> Si se reduce el ancho de banda, podrían aparecer pérdidas y mayores retardos, afectando el rendimiento.

## 4. Visualización en Python

La gráfica muestra que, al no existir pérdida de paquetes ni variaciones de retardo, el rendimiento se mantiene estable y lineal.

**Gráfica:**

![image](https://github.com/user-attachments/assets/88c27b40-9f48-4611-87ba-8b628dba94be)

**Código de visualización:**

```python
import matplotlib.pyplot as plt

# Datos
flows = ['Flow 1', 'Flow 2', 'Flow 3', 'Flow 4']
throughput = [84.9527] * 4  # kbps
delay = [0.0066864] * 4     # segundos
packet_loss = [0] * 4       # %

fig, ax1 = plt.subplots(figsize=(10, 6))

# Throughput
ax1.set_xlabel('Flujos')
ax1.set_ylabel('Throughput (kbps)', color='tab:blue')
ax1.bar(flows, throughput, color='tab:blue', alpha=0.6, label="Throughput (kbps)")
ax1.tick_params(axis='y', labelcolor='tab:blue')

# Delay y Packet Loss
ax2 = ax1.twinx()
ax2.set_ylabel('Delay (s) / Packet Loss (%)', color='tab:red')
ax2.plot(flows, delay, color='tab:red', marker='o', label="Delay (s)")
ax2.plot(flows, packet_loss, color='tab:green', marker='s', label="Packet Loss (%)")
ax2.tick_params(axis='y', labelcolor='tab:red')

plt.title('Desempeño de Flujos de Red')
ax1.legend(loc='upper left')
ax2.legend(loc='upper right')

plt.tight_layout()
plt.show()
```

**Ejemplo con congestión:**

![image](https://github.com/user-attachments/assets/d10a854b-9a3d-4c44-8af0-ebb6754426bc)

## 5. Conclusiones

* Sin pérdida de paquetes bajo condiciones normales de ancho de banda.
* Rendimiento constante y simétrico en todos los flujos.
* Retardo bajo y estable.
* La congestión en la red puede modificar drásticamente el rendimiento y la entrega de datos.
