# Gestion des scènes

Nous pouvons recharger des scènes ou modifier des scènes en utilisant le module de gestion des scènes de Unity.

Par exemple, dans le jeu Pong, nous rechargeons simplement la scène courante à chaque fois que nous perdon

```c#
using UnityEngine;
using UnityEngine.SceneManagement;

public class PongWinUI : MonoBehaviour
{
    public void OnReplay() {
      SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
}
```

Nous avons connecté le bouton « Replay » de l'interface utilisateur à la méthode `OnReplayMethod`. Lorsque le bouton est cliqué, nous rechargeons la scène actuelle.
