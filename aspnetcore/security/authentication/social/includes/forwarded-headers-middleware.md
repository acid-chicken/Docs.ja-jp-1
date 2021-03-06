## <a name="forward-request-information-with-a-proxy-or-load-balancer"></a>転送プロキシ情報を要求またはロード バランサー

プロキシ サーバーまたはロード バランサーの背後にあるアプリを展開する場合、要求ヘッダーでアプリを元の要求情報の一部を転送する可能性があります。 この情報には、セキュリティで保護された要求スキームには、通常が含まれます (`https`)、ホスト、およびクライアントの IP アドレス。 アプリでは、探索し、元の要求情報を使用して、これらの要求ヘッダーを自動的に読み取りますはありません。

スキームは、外部プロバイダーを使用した認証フローに影響するリンクの生成に使用されます。 セキュリティで保護されたスキームが失われる (`https`) が正しくない安全でないリダイレクト Url を生成するアプリで発生します。

元の要求情報を要求の処理用のアプリに使用できるようにするのにには、Forwarded Headers Middleware を使用します。

詳細については、「 <xref:host-and-deploy/proxy-load-balancer> 」を参照してください。
