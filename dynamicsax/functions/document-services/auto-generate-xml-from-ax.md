# Auto Generate XML From AX

![](../../../.gitbook/assets/image.png)

![](../../../.gitbook/assets/image%20%283%29.png)

![](../../../.gitbook/assets/image%20%2811%29.png)

![](../../../.gitbook/assets/image%20%281%29.png)

![](../../../.gitbook/assets/image%20%282%29.png)

```text
static void ExportOrderXml(CIG_ICTradeOrderTable    _tradeOrderTable)
{
    AxdCIG_ICTradeOrder                 TradeOrder;

    AifEntityKey                        key;

    Map                                 map;

    AifPropertyBag                      bag;

    XmlDocument                         xmlDocument = new XmlDocument();

    FileIoPermission                    perm;

    CIG_ICTradeOrderTable                          tradeOrderTable;// = CIG_ICTradeOrderTable::find(_transRefId);

    CIG_ICTradeOrderLine                           tradeOrderLine;



    filename                            filename;// = strfmt("//editest/AX-TEST/%1_%2.xml",strrem(strrem(salesTable.SalesId,'/'),'-'),timenow());

    Boolean                             loadStatus;
    ;

    if(_tradeOrderTable)
    {
        filename = strfmt("//edilive/edi_live/FileSystem/Root/Inbound/ORDER/ORDER_%2.xml",strrem(strrem(_tradeOrderTable.SalesPurchOrderId,'/'),'-'),_tradeOrderTable.EDIGUID);


        perm = new FileIoPermission(fileName,"wr");

        perm.assert();

        map = new Map(Types::Integer, Types::Container);

        map.insert(fieldnum(CIG_ICTradeOrderTable, EDIGUID), [_tradeOrderTable.EDIGUID]);

        map.insert(fieldnum(CIG_ICTradeOrderTable, OrderType), [_tradeOrderTable.OrderType]);

        key = new AifEntityKey();

        key.parmTableId(tablenum(CIG_ICTradeOrderTable));

        key.parmKeyDataMap(map);

        try
        {
            tradeOrder = new AxdCIG_ICTradeOrder();

            loadStatus = xmlDocument.loadXml(tradeOrder.read(key, null, new AifEndPointActionPolicyInfo(), new AifConstraintList(), bag));

            if(loadStatus)
            {
                xmlDocument.save(filename);

                CIG_QueueManager::createQueueManagerData(_tradeOrderTable
                                                        ,xmlDocument.xml()
                                                        ,""
                                                        ,AifMessageDirection::Outbound
                                                        ,filename);

                CIG_TradeInterCompany_EDI::updateStatus(_tradeOrderTable,"Sent");
            }

            CodeAccessPermission::revertAssert();


        }
        catch
        {
            throw error('Error in document service outbound');
        }
    }
}
```
