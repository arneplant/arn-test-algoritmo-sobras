const { app } =require('@azure/functions')

functionconvertirAPorcentaje(valor) {

  if (valor>1) {

    returnvalor/100

  }

  returnvalor

}

functioncalcularSaldosNecesarios(

  saldosCalculados,

  cantidadAFabricar,

  minimo,

  limiteMinimo

) {

  if (saldosCalculados===0) {

    return0

  }

  if (saldosCalculados>=minimo) {

    returnsaldosCalculados

  }

  returnMath.min(

    minimo,

    Math.ceil(cantidadAFabricar*limiteMinimo)

  )

}

functioncalcularPedido(

  configuracion,

  pedidosAnteriores,

  pedido

) {

  constcantidadPedido=Number(pedido.cantidadPedido)

  constcantidadTerminada=Number(pedido.cantidadTerminada??0)

  constcantidadPendiente=Math.max(

    cantidadPedido-cantidadTerminada,

    0

  )

  constacumuladoConAnteriores=pedidosAnteriores.reduce(

    (suma, pedidoAnterior) =>suma+pedidoAnterior.cantidadPendiente,

    0

  )

  constcantidadAFabricar=Math.max(

    acumuladoConAnteriores+

    cantidadPedido-

    configuracion.stock,

    0

  )

  constsaldosCalculados=

    cantidadAFabricar===0

    ?0

    :Math.ceil(

    cantidadAFabricar*

    configuracion.porcentaje

    )

  constsaldosNecesarios=calcularSaldosNecesarios(

    saldosCalculados,

    cantidadAFabricar,

    configuracion.minimo,

    configuracion.limiteMinimo

  )

  constsaldosAsignadosVivosAnteriores=

    pedidosAnteriores.reduce(

    (suma, pedidoAnterior) =>

    suma+pedidoAnterior.saldosAsignadosVivos,

    0

    )

  constsaldosAsignadosPedido=

    saldosNecesarios===0

    ?0

    :Math.max(

    saldosNecesarios-

    saldosAsignadosVivosAnteriores,

    0

    )

  constsaldosAsignadosVivos=

    cantidadPedido===0

    ?0

    :Math.floor(

    (saldosAsignadosPedido/cantidadPedido) *

    cantidadPendiente

    )

  return {

    id:pedido.id??null,

    cantidadPedido,

    cantidadTerminada,

    cantidadPendiente,

    acumuladoConAnteriores,

    cantidadAFabricar,

    saldosCalculados,

    saldosNecesarios,

    saldosAsignadosPedido,

    saldosAsignadosVivos

  }

}

app.http('calcular', {

  methods: ['POST'],

  authLevel:'anonymous',

  route:'calcular',

  handler:async (request, context) => {

    try {

    constbody=awaitrequest.json()

    constconfiguracion= {

    esMaterialCliente:Boolean(body.configuracion.esMaterialCliente),

    minimo:Number(body.configuracion.minimo),

    limiteMinimo:convertirAPorcentaje(

    Number(body.configuracion.limiteMinimo)

    ),

    porcentaje:convertirAPorcentaje(

    Number(body.configuracion.porcentaje)

    ),

    stock:Number(body.configuracion.stock)

    }

    constresultadosPedidosAnteriores= []

    for (constpedidoofbody.pedidosEnCurso) {

    resultadosPedidosAnteriores.push(

    calcularPedido(

    configuracion,

    resultadosPedidosAnteriores,

    pedido

    )

    )

    }

    constresultadoNuevoPedido=calcularPedido(

    configuracion,

    resultadosPedidosAnteriores,

    body.nuevoPedido

    )

    return {

    status:200,

    jsonBody: {

    configuracion,

    pedidosEnCurso:resultadosPedidosAnteriores,

    nuevoPedido:resultadoNuevoPedido

    }

    }

    }

    catch (error) {

    context.error(error)

    return {

    status:500,

    jsonBody: {

    error:error.message

    }

    }

    }

  }

})
