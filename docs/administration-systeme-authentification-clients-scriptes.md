# Authentification des Clients Scriptés

<div class="couleur-introduction">
Pour que les clients scriptés (tels que wget) invoquent des opérations nécessitant une autorisation (telles que la planification d'une compilation), utilisez l'authentification HTTP BASIC pour spécifier le nom d'utilisateur et le jeton API.
</div>

Les versions antérieures de Jenkins vous demandent de spécifier votre mot de passe réel, et cela n'est possible que lorsque votre domaine de sécurité est basé sur un mot de passe (par exemple, les plugins OpenID, Crowd et CAS vous authentifient sans mot de passe, vous n'avez donc tout simplement pas de mot de passe !). La spécification du mot de passe réel est toujours prise en charge, mais elle n'est pas recommandée en raison du risque de divulgation du mot de passe et de la tendance humaine à réutiliser le même mot de passe à différents endroits.

Le jeton API est disponible dans votre page de configuration personnelle. Cliquez sur votre nom dans le coin supérieur droit de chaque page, puis cliquez sur « Configurer » pour voir votre jeton API. (L'URL `$root/me/configure` est un bon raccourci.) Vous pouvez également modifier votre jeton API à partir d'ici.

Notez que Jenkins n'effectue aucune négociation d'autorisation. Autrement dit, il renvoie immédiatement une réponse 403 (Forbidden) au lieu d'une réponse 401 (Unauthorized). Veillez donc à envoyer les informations d'authentification dès la première requête (ce que l'on appelle « l'authentification préventive »).

## Shell avec curl

La commande `curl` est disponible pour la plupart des systèmes d'exploitation, notamment Linux, macOS, Windows, FreeBSD, etc.

``` sh title="SH"
curl -X POST -L --user votre-nom-d'utilisateur:apiToken \
    https://jenkins.example.com/job/your_job/build
```

## Shell avec wget

!!! info " "
    La commande `wget` nécessite l'option `--auth-no-challenge` pour s'authentifier auprès de Jenkins :

``` sh title="SH"
wget --auth-no-challenge \
    --user=user --password=apiToken \
    http://jenkins.example.com/job/your_job/build
```

## Script Groovy utilisant cdancy/jenkins-rest

Le [client cdancy/jenkins-rest](https://github.com/cdancy/jenkins-rest) simplifie considérablement l'accès à l'API REST. Le code Groovy suivant montre comment s'authentifier auprès de Jenkins et obtenir certaines informations système :

``` groovy title="GROOVY"
@Grab(group=“com.cdancy”, module=“jenkins-rest”, version=“0.0.18”)

import com.cdancy.jenkins.rest.JenkinsClient

JenkinsClient client = JenkinsClient.builder()
    .endPoint(« http://127.0.0.1:8080 ») // Facultatif. La valeur par défaut est http://127.0.0.1:8080.
    .credentials(« user:apiToken ») // Facultatif.
    .build()

println(client.api().systemApi().systemInfo())
```

Pour plus d'informations, consultez le [wiki cdancy/jenkins-rest](https://github.com/cdancy/jenkins-rest/wiki).

## Exemple de script typescript utilisant axios

Le code suivant montre comment créer un travail :

``` typescript title="TYPESCRIPT"
const credentials = `myusername:myapitoken`;
const baseJenkinsUrl = "http://myjenkins:8080";
async function getCrumb() {
    return axios({
      method: "get",
      url: baseJenkinsUrl + "/crumbIssuer/api/json",
      withCredentials: false,
    });
  }
getCrumb().then((response: any) => {
      let crumb = response.data;
     const base64Credentials = btoa(credentials);
    const authHeader = `Basic ${base64Credentials}`;
    // Create new job
    const requestConfig: AxiosRequestConfig = {
      method: "post",
      url: `${baseJenkinsUrl}/createItem?name=${jobName}`,
      headers: {
        Authorization: authHeader,
        [crumb.crumbRequestField]: crumb.crumb,
        "Content-Type": "application/xml",
      },
      data: xmlconfig,
    };
    axios(requestConfig);
});
```

## Exemple Perl LWP pour un client scripté

L'exemple Perl suivant utilise le module LWP pour démarrer une tâche via un jeton « Trigger builds remotely » (Déclencher des builds à distance) :

``` perl title="PERL"
#
# Utiliser LWP pour exécuter une tâche Jenkins
# Définir authorization_basic sur l'objet de requête
# pour utiliser l'autorisation HTTP BASIC, apparemment
# en gérant déjà correctement la partie préemptive de cette
# manière.
#
use strict;
use warnings;

use LWP;

my $server = “srvname”;
my $srvurl = « http://$server/jenkins »;
my $uagent = LWP::UserAgent->new;
my $req = HTTP::Request->new(
  GET => « $srvurl/job/test/build?token=theTokenConfiguredForThisJob&cause=LWP+Test »
);
$req->authorization_basic(“username@mydomain.com”, “apiToken”);
my $res = $uagent->request($req);

# Vérifier le résultat de la réponse
print « Résultat : » . $res->status_line . « \n »;
print $res->headers->as_string;
print « \n »;
if (!$res->is_success) {
  print « Échec\n »;
}
else {
  print « Succès !\n »;
  # print $res->content, « \n »;
}
```

## Exemple Java avec httpclient 4.3.x

Cela entraînera l'émission préventive d'une authentification par httpclient 4.3 :

``` java title="JAVA"
import java.io.IOException;
import java.net.URI;

import org.apache.http.HttpHost;
import org.apache.http.HttpResponse;
import org.apache.http.auth.AuthScope;
import org.apache. http.auth.UsernamePasswordCredentials;
import org.apache.http.client.AuthCache;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.CredentialsProvider;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.protocol.HttpClientContext;
import org.apache.http.impl.auth.BasicScheme;
import org.apache.http.impl.client.BasicAuthCache;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache. http.util.EntityUtils ;

public class JenkinsScraper {

    public String scrape(String urlString, String username, String password)
        throws ClientProtocolException, IOException {
        URI uri = URI.create(urlString) ;
        HttpHost host = new HttpHost(uri.getHost(), uri.getPort(), uri.getScheme()) ;
        CredentialsProvider credsProvider = new BasicCredentialsProvider();
        credsProvider.setCredentials(new AuthScope(uri.getHost(), uri.getPort()),
            new UsernamePasswordCredentials(username, password));
        // Créer une instance AuthCache
        AuthCache authCache = new BasicAuthCache();
        // Générer un objet BASIC scheme et l'ajouter au cache d'authentification local
        BasicScheme basicAuth = new BasicScheme();
        authCache.put(host, basicAuth);
        CloseableHttpClient httpClient =
            HttpClients.custom().setDefaultCredentialsProvider(credsProvider).build();
        HttpGet httpGet = new HttpGet(uri);
        // Ajouter AuthCache au contexte d'exécution
        HttpClientContext localContext = HttpClientContext.create();
        localContext.setAuthCache(authCache);

        HttpResponse response = httpClient.execute(host, httpGet, localContext);

        return EntityUtils.toString(response.getEntity());
    }

}
```

