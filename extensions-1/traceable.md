# Traceable

The traceable extension, when decorating a tez value, generates an dedicated contract transaction to load the corresponding amount.

{% code-tabs %}
{% code-tabs-item title="traceable.arlx" %}
```ocaml
archetype extension traceable (

   variable[%traceable] anamount tez from fromR to toR

) = {

  action transfer_<%anamount> = {
    called by fromR
    (* accept transfer *)
    require {
      c1 : transferred = anamount
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

