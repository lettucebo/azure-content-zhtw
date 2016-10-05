## 服務鏈結 - 透過已對等互連的 VNet 進行傳輸

雖然使用系統路由可自動加速您部署的流量，但是也有一些您希望透過虛擬設備控制封包路由的情況。在此案例中，訂用帳戶中有兩個 VNet，分別是 HubVNet 和 VNet1，如下圖所示。在 VNet HubVNet 中部署網路虛擬設備 (NVA)。在 HubVNet 和 VNet1 之間建立 VNet 對等互連後，您可以設定使用者定義路徑並在 HubVNet 中將 NVA 指定為下一個躍點。

![NVA 傳輸](./media/virtual-networks-create-vnetpeering-scenario-transit-include/figure01.PNG)

> [AZURE.NOTE] 為了簡單起見，假設此處所有 Vnet 都位於同一訂用帳戶中。但跨訂用帳戶的案例也適用。

啟用傳輸路由的關鍵屬性是「允許轉送流量」參數。可在對等互連 VNet 中允許接收來自 NVA 和傳送至 NVA 的流量。

<!---HONumber=AcomDC_0928_2016-->