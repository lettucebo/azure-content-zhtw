| | **點對站** | **網站間** | **ExpressRoute** |
|------------------------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| **Azure 支援的服務** | 雲端服務及虛擬機器 | 雲端服務及虛擬機器 | [服務清單](../expressroute/expressroute-faqs.md#supported-services) |
| **典型的頻寬** | 彙總通常 < 100 Mbps | 彙總通常 < 100 Mbps | 50 Mbps、100 Mbps、200 Mbps、500 Mbps、1 Gbps、2 Gbps、5 Gbps、10 Gbps |
| **支援的通訊協定** | 安全通訊端通道通訊協定 (SSTP) | IPsec | 透過 VLAN、NSP 的 VPN 技術 (MPLS、VPLS、...) 的直接連接 |
| **路由** | RouteBased (動態) | 我們支援 PolicyBased (靜態路由) 和 RouteBased (動態路由) VPN | BGP |
| **連接恢復功能** | 主動-被動 | 主動-被動 | 主動-主動 |
| **典型的使用案例** | 原型設計、雲端服務和虛擬機器的開發人員/測試/實驗室案例 | 雲端服務和虛擬機器的開發/測試/實驗室案例和小規模生產工作負載 | 所有 Azure 服務 (已驗證的清單)、企業層級與關鍵性工作負載、備份、巨量資料、Azure 做為 DR 網站的存取 |
| **SLA** | [SLA](https://azure.microsoft.com/support/legal/sla/) | [SLA](https://azure.microsoft.com/support/legal/sla/) | [SLA](https://azure.microsoft.com/support/legal/sla/) |
| **定價** | [定價](https://azure.microsoft.com/pricing/details/vpn-gateway/) | [定價](https://azure.microsoft.com/pricing/details/vpn-gateway/) | [定價](https://azure.microsoft.com/pricing/details/expressroute/) |
| **技術文件** | [VPN 閘道文件](https://azure.microsoft.com/documentation/services/vpn-gateway/) | [VPN 閘道文件](https://azure.microsoft.com/documentation/services/vpn-gateway/) | [ExpressRoute 文件](https://azure.microsoft.com/documentation/services/expressroute/) |
| **常見問題集** | [VPN 閘道常見問題集](vpn-gateway-vpn-faq.md) | [VPN 閘道常見問題集](vpn-gateway-vpn-faq.md) | [ExpressRoute 常見問題集](../expressroute/expressroute-faqs.md) |

<!---HONumber=AcomDC_0921_2016-->